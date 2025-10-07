# app.py
import uuid
import subprocess
import asyncio
import signal
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from pydantic import BaseModel
from google.adk.agents import Agent
from google.adk.mcp import MCPToolset

app = FastAPI()
sessions = {}

class ChatRequest(BaseModel):
    message: str

def start_serena_server(project_path: str = None):
    """
    Launch a new Serena MCP server via uvx with optional project context.
    """
    cmd = [
        "uvx", "--from", "git+https://github.com/oraios/serena",
        "serena", "start-mcp-server", "--transport", "stdio"
    ]
    if project_path:
        cmd += ["--project", project_path]

    proc = subprocess.Popen(
        cmd,
        stdin=subprocess.PIPE,
        stdout=subprocess.PIPE,
        stderr=subprocess.PIPE,
        text=True
    )
    return proc, cmd

@app.post("/session/start")
async def start_session(project_path: str = None):
    session_id = str(uuid.uuid4())
    proc, cmd = start_serena_server(project_path)

    mcp_toolset = MCPToolset.stdio(command=cmd)
    agent = Agent(
        name=f"agent_{session_id}",
        model="gemini-1.5-flash",
        instruction="You are a session-isolated coding assistant using Serena MCP.",
        tools=[mcp_toolset]
    )

    sessions[session_id] = {
        "proc": proc,
        "agent": agent,
        "last_active": asyncio.get_event_loop().time()
    }
    return {"session_id": session_id}

@app.post("/session/{session_id}/chat")
async def chat(session_id: str, req: ChatRequest):
    session = sessions.get(session_id)
    if not session:
        return {"error": "Invalid session"}
    agent = session["agent"]
    session["last_active"] = asyncio.get_event_loop().time()
    response = await agent.run_async(req.message)
    return {"response": response}

@app.websocket("/session/{session_id}/ws")
async def chat_ws(websocket: WebSocket, session_id: str):
    await websocket.accept()
    session = sessions.get(session_id)
    if not session:
        await websocket.send_json({"error": "Invalid session"})
        await websocket.close()
        return

    agent = session["agent"]
    try:
        while True:
            data = await websocket.receive_text()
            session["last_active"] = asyncio.get_event_loop().time()
            async for chunk in agent.run_stream_async(data):
                await websocket.send_text(chunk)
    except WebSocketDisconnect:
        pass

@app.delete("/session/{session_id}")
async def end_session(session_id: str):
    session = sessions.pop(session_id, None)
    if session:
        session["proc"].terminate()
        return {"status": "terminated"}
    return {"error": "Invalid session"}

# Background task: cleanup idle sessions
async def cleanup_sessions(timeout: int = 600):
    while True:
        now = asyncio.get_event_loop().time()
        for sid, session in list(sessions.items()):
            if now - session["last_active"] > timeout:
                session["proc"].terminate()
                sessions.pop(sid, None)
        await asyncio.sleep(60)

@app.on_event("startup")
async def startup_event():
    asyncio.create_task(cleanup_sessions())
