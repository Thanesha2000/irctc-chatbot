FROM python:3.11-slim

WORKDIR /app

# Copy requirements from the agents subdirectory relative to the context (back/)
COPY agents/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy the entire backend directory (agents, data, vectorstore)
COPY . .

# Move into the agents directory so that relative imports and data folder paths work correctly
WORKDIR /app/agents

EXPOSE 7860

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "7860"]
