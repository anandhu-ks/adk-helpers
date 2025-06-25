from google.genai.types import Content, Part

async def call_agent_async(runner, user_id, session_id, user_input):
    user_content = Content(parts=[Part(text=user_input)], role="user")
    async for event in runner.run_async(user_id=user_id, session_id=session_id, new_message=user_content):
        if event.is_final_response() and event.content and event.content.parts:
            print("Assistant:", event.content.parts[0].text)




import asyncio
from dotenv import load_dotenv
from google.adk.runners import Runner
from google.adk.sessions import DatabaseSessionService
from smart_todo.agent import root_agent
from smart_todo.utils import call_agent_async

load_dotenv()

db_url = "mysql+pymysql://root:@localhost:3306/adk"

async def main_async():
    APP_NAME = "Smart Todo"
    USER_ID = "AgentUser"  # Use a fixed user ID for simplicity
    STATE = {"user_id": USER_ID}

    session_service = DatabaseSessionService(db_url=db_url)

    # ✅ Always create a new session with state
    new_session = await session_service.create_session(
        app_name=APP_NAME,
        user_id=USER_ID,
        state=STATE,
    )
    SESSION_ID = new_session.id
    print(f"Created new session: {SESSION_ID}")

    runner = Runner(
        agent=root_agent,
        app_name=APP_NAME,
        session_service=session_service,
    )

    print("\nWelcome to your Personal Assistant!")
    print("Type 'exit' or 'quit' to end the conversation.\n")

    while True:
        user_input = input("You: ")
        if user_input.lower() in ["exit", "quit"]:
            print("Ending conversation. Your data has been saved.")
            break
        await call_agent_async(runner, USER_ID, SESSION_ID, user_input)

if __name__ == "__main__":
    asyncio.run(main_async())
