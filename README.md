# adk_manager.py
import asyncio
import subprocess
import logging
from contextlib import asynccontextmanager
from utils import find_available_port

logging.basicConfig(level=logging.INFO)

@asynccontextmanager
async def mcp_server_process(server_id: int):
    """
    Manages the lifecycle of a persistent MCP server process on a unique port.
    """
    port = find_available_port()
    url = f"http://localhost:{port}"
    # The --port argument is passed to the start-mcp-server command
    command = [
        "uvx",
        "--from", "git+https://github.com/oraios/serena",
        "serena",
        "start-mcp-server",
        "--port", str(port)
    ]
    process = None
    try:
        logging.info(f"Agent {server_id}: Starting MCP server on {url}...")
        process = subprocess.Popen(
            command,
            stdout=subprocess.PIPE,
            stderr=subprocess.STDOUT,  # Capture both stdout and stderr
            text=True,
        )

        # Wait for the server to start and print output
        started = False
        timeout = 20
        while not started and timeout > 0:
            output = process.stdout.readline()
            if "server has started" in output.lower():
                started = True
                logging.info(f"Agent {server_id}: Server started successfully.")
            timeout -= 1
            await asyncio.sleep(1)
            
        if not started:
            raise RuntimeError(f"Agent {server_id}: MCP server failed to start on time.")
        
        yield url

    except Exception as e:
        logging.error(f"Agent {server_id}: Failed to start MCP server: {e}")
        raise
    finally:
        if process and process.poll() is None:
            logging.info(f"Agent {server_id}: Terminating MCP server process.")
            process.terminate()
            process.wait()
            logging.info(f"Agent {server_id}: MCP server process terminated.")



# agents.py
import adk
import asyncio
from adk.mcp.tools import MCPToolset
from adk.mcp.types import StreamableHttpServerParameters
from adk.tools import LlmAgent, tool
from adk_manager import mcp_server_process
import logging

logging.basicConfig(level=logging.INFO)

# Define a tool for each agent
@tool(name="internal_info", description="Provides information about the ADK agent.")
def internal_info(query: str) -> str:
    return "This ADK agent is configured with its own dedicated Serena MCP server."

async def create_agent(server_id: int):
    """Creates a single agent with its own persistent MCP server."""
    async with mcp_server_process(server_id) as server_url:
        mcp_toolset = MCPToolset(
            connection_params=StreamableHttpServerParameters(url=server_url),
            name=f"serena_agent_{server_id}",
            description=f"A dedicated Serena MCP server for agent {server_id}.",
        )
        agent = LlmAgent(
            tools=[internal_info, mcp_toolset],
            instruction=f"You are Agent {server_id}. You can use the 'internal_info' tool or the 'serena_agent_{server_id}' tool. The serena server is running persistently in the background.",
            model="gemini-1.5-flash",
        )
        
        logging.info(f"Agent {server_id}: Starting ADK web interface...")
        await adk.run_web(agent)


async def main():
    """Runs multiple agents with their own servers concurrently."""
    num_agents = 2  # Number of agents to run
    tasks = [create_agent(i) for i in range(num_agents)]
    await asyncio.gather(*tasks)


if __name__ == '__main__':
    asyncio.run(main())
