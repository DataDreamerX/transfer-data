from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field
from typing import Dict, Any, Optional
import uuid
import uvicorn
import asyncio
from contextlib import asynccontextmanager

# ADK Imports
from google.adk.agents import LlmAgent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai import types

# --- Global State Management ---
# Store for active Runners, keyed by a unique Agent ID (UUID).
# We use InMemorySessionService for simplicity and true isolation, 
# but sessions will be lost if the server restarts.
active_runners: Dict[str, Runner] = {}
session_service = InMemorySessionService() 

# --- Pydantic Models for API ---

class AgentCreationRequest(BaseModel):
    """Schema for creating a new agent."""
    name: str = Field(..., description="A unique name for the agent (e.g., 'TravelBot').")
    instruction: str = Field(..., description="The system instruction defining the agent's personality and role.")
    model: str = Field("gemini-2.5-flash", description="The LLM model to power the agent.")

class AgentCreationResponse(BaseModel):
    """Schema for the new agent ID."""
    agent_id: str
    message: str

class ChatRequest(BaseModel):
    """Schema for sending a message to an agent."""
    agent_id: str = Field(..., description="The ID of the dynamically created agent.")
    user_id: str = Field(..., description="A unique ID for the user initiating the conversation.")
    message: str = Field(..., description="The user's prompt or message.")
    session_id: Optional[str] = Field(None, description="The session ID to continue an existing chat. Leave blank for the first message.")

class ChatResponse(BaseModel):
    """Schema for the agent's response."""
    session_id: str
    response: str
    is_new_session: bool
    agent_name: str


# --- FastAPI Application ---

@asynccontextmanager
async def lifespan(app: FastAPI):
    """
    Startup/Shutdown event to ensure the SessionService is initialized.
    Although InMemory, the ADK framework requires this structure.
    """
    print("ADK Session Service starting up...")
    await session_service.startup()
    yield 
    print("ADK Session Service shutting down...")
    await session_service.shutdown()

app = FastAPI(
    title="ADK Dynamic Agent API",
    description="API to dynamically create and chat with isolated ADK agents.",
    version="1.0.0",
    lifespan=lifespan
)

# --- Endpoints ---

@app.post("/agents", response_model=AgentCreationResponse)
async def create_new_agent(request: AgentCreationRequest):
    """
    Endpoint to dynamically create a new, isolated ADK agent instance and its Runner.
    """
    agent_id = str(uuid.uuid4())
    
    try:
        # 1. Dynamically create the ADK Agent
        new_agent = LlmAgent(
            name=request.name,
            model=request.model,
            instruction=request.instruction,
            tools=[] 
        )
        
        # 2. Create an ADK Runner for this agent instance
        new_runner = Runner(
            agent=new_agent,
            app_name=request.name, # Use agent name as app_name
            session_service=session_service
        )
        
        # 3. Store the Runner instance for later use
        active_runners[agent_id] = new_runner
        
        return AgentCreationResponse(
            agent_id=agent_id,
            message=f"Agent '{request.name}' created successfully. Use this ID for chat."
        )
        
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Failed to create agent: {e}")


@app.post("/chat", response_model=ChatResponse)
async def chat_with_agent(request: ChatRequest):
    """
    Endpoint to send a message to an existing, isolated agent, 
    maintaining the session history.
    """
    
    # 1. Retrieve the active Runner
    runner = active_runners.get(request.agent_id)
    if not runner:
        raise HTTPException(status_code=404, detail=f"Agent ID '{request.agent_id}' not found.")
        
    is_new_session = False
    
    try:
        # 2. Get or Create the Session
        app_name = runner.app_name
        
        if request.session_id:
            # Try to resume existing session
            session = await session_service.get_session(
                app_name=app_name, 
                user_id=request.user_id, 
                session_id=request.session_id
            )
        else:
            # Create a new session with the user_id
            session = await session_service.create_session(
                app_name=app_name,
                user_id=request.user_id
            )
            is_new_session = True

        # 3. Prepare the ADK Content object
        content = types.Content(
            role="user",
            parts=[types.Part(text=request.message)]
        )
        
        # 4. Run the Agent (ADK handles memory/history via the session_id)
        result = await runner.run(content=content, session_id=session.id)
        
        # 5. Extract the final agent response
        agent_response_parts = result.output.parts
        final_response = agent_response_parts[-1].text if agent_response_parts else "No response from agent."
        
        return ChatResponse(
            session_id=session.id,
            response=final_response,
            is_new_session=is_new_session,
            agent_name=runner.agent.name
        )
        
    except Exception as e:
        # In a production app, use proper logging here.
        print(f"Error processing chat request for agent {request.agent_id}: {e}")
        raise HTTPException(status_code=500, detail="Internal error during agent conversation.")

if __name__ == "__main__":
    # --- Instructions to Run ---
    # 1. Make sure you have the required dependencies: pip install -r requirements.txt
    # 2. Set your Google API key as an environment variable (or configure ADK credentials).
    # 3. Run the server: 
    print("Starting FastAPI server...")
    uvicorn.run(app, host="0.0.0.0", port=8000)
