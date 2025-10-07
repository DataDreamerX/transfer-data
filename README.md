# app.py
import uuid
import subprocess
from fastapi import FastAPI
from pydantic import BaseModel
from google.adk.agents import Agent
from google.adk.mcp import MCPToolset

app = FastAPI()
sessions = {}

class ChatRequest(BaseModel):
    message: str

def start_serena_server():
    """
    Launch a new Serena MCP server via uvx.
    Using stdio transport so ADK can connect directly.
    """
    proc = subprocess.Popen(
        [
            "uvx", "--from", "git+https://github.com/oraios/serena",
            "serena", "start-mcp-server", "--transport", "stdio"
        ],
        stdin=subprocess.PIPE,
        stdout=subprocess.PIPE,
        text=True
    )
    return proc

@app.post("/session/start")
async def start_session():
    session_id = str(uuid.uuid4())
    proc = start_serena_server()

    # Connect ADK agent to this Serena MCP server
    mcp_toolset = MCPToolset.stdio(
        command=[
            "uvx", "--from", "git+https://github.com/oraios/serena",
            "serena", "start-mcp-server", "--transport", "stdio"
        ]
    )

    agent = Agent(
        name=f"agent_{session_id}",
        model="gemini-1.5-flash",
        instruction="You are a session-isolated coding assistant using Serena MCP.",
        tools=[mcp_toolset]
    )

    sessions[session_id] = {"proc": proc, "agent": agent}
    return {"session_id": session_id}

@app.post("/session/{session_id}/chat")
async def chat(session_id: str, req: ChatRequest):
    session = sessions.get(session_id)
    if not session:
        return {"error": "Invalid session"}
    agent = session["agent"]
    response = await agent.run_async(req.message)
    return {"response": response}

@app.delete("/session/{session_id}")
async def end_session(session_id: str):
    session = sessions.pop(session_id, None)
    if session:
        session["proc"].terminate()
        return {"status": "terminated"}
    return {"error": "Invalid session"}
