# Technical Product Requirements Document: Isek MVP

## 1. Product Overview
Isek is a decentralized agent network framework for building, deploying, and managing intelligent, collaborative agent-to-agent (A2A) systems. It provides the core infrastructure for AI agents to autonomously discover peers, share context, and cooperatively solve tasks in a distributed, self-organizing fashion without a central orchestrator.

## 2. User Stories

**Persona: AI Developer**
1.  **Given** I am an AI developer, **when** I want to create a new autonomous agent, **then** I should be able to define its identity, core model, and instructions using the `IsekAgent` Python class.
2.  **Given** I have an `IsekAgent`, **when** I provide it with a task, **then** it should execute the task using its configured LLM and return a response.
3.  **Given** I want my agent to have access to external functions, **when** I define a `Toolkit` with custom Python functions, **then** I should be able to attach it to my `IsekAgent`.
4.  **Given** my agent needs to remember past interactions, **when** I attach a `Memory` module to it, **then** the agent should persist and recall conversation history.
5.  **Given** I want to use a different language model, **when** I create a new model adapter inheriting from `BaseAdapter`, **then** I should be able to plug it into my `IsekAgent`.
6.  **Given** I need to build a workflow with multiple specialized agents, **when** I define several `IsekAgent` instances, **then** I should be able to group them into an `IsekTeam`.
7.  **Given** I have an `IsekTeam`, **when** I set the mode to `sequential`, **then** the output of one agent should be passed as the input to the next agent in the team.
8.  **Given** I have an `IsekTeam`, **when** I set the mode to `coordinate`, **then** a higher-level LLM call should be used to synthesize a final response based on the capabilities of all team members.
9.  **Given** I want to quickly test the framework, **when** I use the `isek example list` CLI command, **then** I should see a list of available example scripts.
10. **Given** I have chosen an example, **when** I run `isek example run lv5_team_agent`, **then** the corresponding Python script should execute.

**Persona: System Operator**
11. **Given** I want to make my agents available on the network, **when** I instantiate a `Node` object in Python, **then** I should be able to start a server process that hosts my agents.
12. **Given** I am running a `Node`, **when** it starts, **then** it should also launch and manage the associated Node.js `p2p_server.js` process for network discovery.
13. **Given** my node is running, **when** another node joins the network, **then** my node should become aware of it through the registry.
14. **Given** I need a robust service discovery mechanism for a production environment, **when** I configure the `Node` to use the `EtcdRegistry`, **then** all nodes should register themselves with and discover peers from the shared etcd cluster.
15. **Given** a node is running, **when** I check its status, **then** it should periodically send a heartbeat to the registry to signal it is still alive.
16. **Given** I want to run a node as a background service, **when** I call `node.build_server(daemon=True)`, **then** the node's API server should run in a non-blocking background thread.

**Persona: End User / Researcher**
17. **Given** an agent is running on a node, **when** I want to interact with it, **then** I should be able to send it a message via an HTTP request to the node's API.
18. **Given** two agents are running on different nodes, **when** one agent wants to communicate with the other, **then** it should use the `node.send_message()` method to route the message through the A2A protocol.
19. **Given** the P2P network is enabled, **when** an agent sends a message to another agent, **then** the communication should be routed through the `libp2p` relay.
20. **Given** a developer has exposed an agent via the Chainlit UI, **when** I open the web interface, **then** I should see a chat window to interact with the agent.
21. **Given** I am chatting with an agent via the Chainlit UI, **when** I send a message, **then** the agent's response should appear in the chat history.
22. **Given** I need to set up the environment for the first time, **when** I run `isek setup`, **then** the CLI should install both Python and required Node.js dependencies.

## 3. User Flows

**Flow 1: Creating and Running a Single Agent Locally**
1.  Developer creates a new Python file (`my_agent.py`).
2.  Developer adds `OPENAI_API_KEY` to a `.env` file.
3.  In `my_agent.py`, the developer imports `IsekAgent` and `OpenAIModel`.
4.  The developer instantiates the `IsekAgent`, providing a name, description, and the `OpenAIModel`.
5.  The developer calls the `agent.run("some task")` method.
6.  The agent class calls the `OpenAIModel`, which makes an API request to OpenAI.
7.  The response is returned to the script and printed to the console.

**Flow 2: Setting up a Distributed Agent Network**
1.  Operator starts an `etcd` instance for service discovery.
2.  On Machine A, a developer creates `node_a.py`.
3.  In `node_a.py`, they define `AgentA` and instantiate a `Node` configured with the `EtcdRegistry` address. They start the node server via `node.build_server()`.
4.  The Node server starts, launches the `p2p_server.js` sidecar, registers itself in etcd, and begins sending heartbeats.
5.  On Machine B, a developer creates `node_b.py` with `AgentB` and a `Node` configured identically. They start this node.
6.  Node B registers in etcd. Through the heartbeat mechanism, both nodes refresh their local list of available peers and discover each other.
7.  In `node_b.py`, the developer calls `node_b.send_message(receiver_node_id="node_a_id", message="Hello from Agent B")`.
8.  The message is sent via an HTTP request from Node B to Node A's server, completing the A2A communication.

**Flow 3: Interacting with an Agent via the UI**
1.  A developer has the `UI/chainlit/chainlit_app.py` file ready.
2.  The developer runs `chainlit run UI/chainlit/chainlit_app.py -w` from the terminal.
3.  A web server starts, and a browser window opens to the Chainlit UI.
4.  The UI displays a welcome message from the agent as defined in the `@cl.on_chat_start` function.
5.  The user types "Hello, who are you?" into the input box and presses enter.
6.  The `@cl.on_message` function is triggered, which calls `agent.run()` with the user's message.
7.  The agent's response is streamed back to the UI and displayed as a new message in the chat history.

## 4. Screens and UI/UX

For the MVP, the primary UI is a command-line interface and a sample web application.

**1. Command-Line Interface (CLI)**
*   **Description:** The main interface for developers and operators to manage the Isek environment.
*   **UI Elements:** Standard command-line text. Uses `click` for formatting, with colored output for status messages (green for success, red for errors).
*   **Interactions:**
    *   `isek setup`: Installs dependencies.
    *   `isek example list`: Shows available examples.
    *   `isek example run <name>`: Executes an example.
    *   `isek registry`: Starts a local development registry.
    *   `isek clean`: Removes temporary files.

**2. Chainlit Web UI (Example Application)**
*   **Description:** A simple, clean chat interface for direct interaction with a single `IsekAgent`.
*   **UI Elements:**
    *   **Chat Window:** A vertically scrolling log of the conversation history. User messages are aligned on one side, and agent responses on the other.
    *   **Message Input Box:** A text area at the bottom of the screen for the user to type their messages.
    *   **Send Button:** A button to submit the message.
*   **Style:** Inherits the default modern and clean aesthetic of the Chainlit framework.

## 5. Features and Functionality

*   **Agent Core:** Python class (`IsekAgent`) allowing definition of agent properties (name, model, tools, memory, instructions).
*   **Team Orchestration:** Python class (`IsekTeam`) to manage groups of agents with multiple coordination modes (sequential, coordinate, route).
*   **Pluggable Models:** An adapter pattern (`isek.models.base.Model`) allows for integrating various LLMs (OpenAI and LiteLLM are provided).
*   **Tool Integration:** A `Toolkit` system allows agents to be equipped with standard (e.g., calculator) or custom Python functions.
*   **Stateful Memory:** A `Memory` module can be attached to agents to maintain conversation history.
*   **Node Server:** A `fastapi`-based server (`isek.node.node_v2.Node`) that hosts agents and exposes them to the network via an HTTP API.
*   **P2P Networking Layer:** A Node.js sidecar process (`p2p_server.js`) using `libp2p` to handle peer discovery and establish P2P connections, primarily through a public relay node.
*   **Service Registry:** A pluggable system (`isek.node.registry.Registry`) for nodes to register themselves and discover others. A local in-memory `DefaultRegistry` and a production-ready `EtcdRegistry` are provided.
*   **Management CLI:** A `click`-based command-line tool (`isek`) for managing the development environment and running examples.

## 6. Technical Architecture

The Isek framework employs a hybrid, multi-process architecture.

*   **Application Layer (Python):** This is the primary layer where developers define and interact with agents, teams, and nodes. It contains all the core logic for agent behavior, model interaction, and tool execution.
*   **Node Server Layer (Python/FastAPI):** Each `Node` runs as a FastAPI application, serving an HTTP API for agent interaction and internode communication. This layer acts as the bridge between the agent logic and the network.
*   **Networking/Discovery Layer (Node.js/libp2p):** For decentralized discovery and communication, each Python node spawns and manages a `p2p_server.js` sidecar process. The Python node communicates with this sidecar via local HTTP requests. The Node.js process connects to a public `libp2p` relay, announces its presence, and can then establish P2P connections with other nodes.
*   **Service Discovery Layer (Pluggable):** Node addresses and metadata are stored in a registry. For local development, this is a simple in-memory dictionary. For distributed setups, `etcd` is used as a centralized, reliable key-value store for service discovery.

![Architecture Diagram](https://i.imgur.com/example.png) (Conceptual placeholder for a diagram showing Python Node -> Node.js Sidecar -> Libp2p Relay -> Other Nodes)

## 7. System Design

*   **IsekAgent Process:**
    1.  An `IsekAgent` is instantiated in a Python script.
    2.  When `agent.run(task)` is called, the agent constructs a prompt using its instructions and the task.
    3.  It passes the prompt to its configured `Model` adapter.
    4.  The adapter handles the API call to the external LLM.
    5.  If the LLM requests a tool, the `Model` adapter invokes the tool from the agent's `Toolkit`, gets the result, and sends it back to the LLM to get the final response.
    6.  The final response is returned.
*   **Isek Node and P2P Communication Process:**
    1.  An `Isek Node` is started. It launches a `uvicorn` server for its FastAPI app.
    2.  The Node also executes `node isek/protocol/p2p/p2p_server.js --port=<p2p_port> --agent_port=<node_port>` as a subprocess.
    3.  The `p2p_server.js` process initializes `libp2p`, connects to a public relay, and gets a peer ID and a routable P2P address. It then listens for commands from its parent Python process via a local Express.js server.
    4.  The Python Node registers its own details (HTTP URL, P2P address) in the configured `Registry`.
    5.  A periodic `__bootstrap_heartbeat` function in the Python Node keeps its registry entry alive and fetches an updated list of all other nodes.
    6.  When `node.send_message()` is called to a peer, the node looks up the peer's P2P address from its local cache and sends the message payload to its own `p2p_server.js` sidecar, which then forwards it over the `libp2p` network.

## 8. API Specifications

The `Isek Node` (FastAPI server) exposes a simple internal API. The primary interaction endpoint is the root path, which is used by the P2P server sidecar.

**Endpoint: `POST /`**
*   **Purpose:** The main entry point for receiving messages from other agents, typically forwarded by the local P2P server.
*   **Request Body:** A JSON object representing the message payload. The exact structure is defined by the `A2AProtocol`.
    ```json
    {
      "sender_node_id": "string",
      "message": "string",
      // ... other protocol fields
    }
    ```
*   **Response Body:**
    ```json
    {
      "status": "received",
      "response": "string" // The receiving agent's reply
    }
    ```

The `p2p_server.js` (Express.js server) exposes an API for the Python process to control it.

**Endpoint: `GET /p2p_context`**
*   **Purpose:** For the Python node to retrieve the `peer_id` and `p2p_address` after the libp2p node has initialized.
*   **Response Body:**
    ```json
    {
      "peer_id": "12D3Koo...",
      "p2p_address": "/ip4/..."
    }
    ```

**Endpoint: `POST /call_peer?p2p_address=<address>`**
*   **Purpose:** For the Python node to send a message to a remote peer.
*   **Request Body:** The message payload.
*   **Response Body:** The reply from the remote peer.

## 9. Data Model

*   **AgentCard:**
    *   `name`: string
    *   `description`: string
    *   `capabilities`: List[string] (e.g., "memory", "tools")
    *   `tools`: List[dict] (List of available tool function schemas)
    *   `model_type`: string (e.g., "OpenAIModel")
*   **NodeDetails:**
    *   `node_id`: string (UUID)
    *   `host`: string
    *   `port`: int
    *   `metadata`: dict (Contains the `AgentCard` and P2P addressing info)
*   **Message:** (Conceptual)
    *   `sender_id`: string
    *   `receiver_id`: string
    *   `session_id`: string
    *   `content`: string
    *   `timestamp`: datetime

## 10. Security Considerations

*   **API Keys:** All interactions with external LLMs require API keys. The framework relies on `.env` files for key management. Keys should not be hardcoded or committed to source control.
*   **Network Security:** Inter-node communication via the default HTTP protocol is unencrypted. For production use, nodes should be deployed behind a reverse proxy (e.g., Nginx) that provides TLS encryption.
*   **P2P Security:** The `libp2p` layer uses the `noise` protocol for encrypted connections between peers.
*   **Tool Execution:** Agents can be equipped with tools that execute arbitrary code. Only trusted tools should be provided to agents, as a malicious LLM could potentially exploit a tool to perform harmful actions.
*   **Authentication:** The MVP has no built-in authentication for the node API. Access should be restricted at the network level.

## 11. Performance Requirements

*   **A2A Latency:** Message passing time between two agents on different nodes should be under 500ms within the same geographic region, excluding LLM processing time.
*   **Node Capacity:** A single standard Node instance should be able to host at least 10-20 idle agents without significant performance degradation.
*   **CLI Responsiveness:** All CLI commands should execute in under 2 seconds.

## 12. Scalability Considerations

*   **Horizontal Scaling:** The system is designed to scale horizontally. System capacity can be increased by adding more `Isek Node` instances, each running on its own hardware.
*   **Registry Scalability:** The default in-memory registry is a single point of failure and does not scale. For multi-node deployments, the `EtcdRegistry` provides a distributed, fault-tolerant service discovery mechanism that can scale to thousands of nodes.
*   **P2P Relay:** The public `libp2p` relay node can become a bottleneck. For large-scale deployments, a dedicated, private relay infrastructure would be required.

## 13. Testing Strategy

*   **Unit Tests:** Use `pytest` to test individual classes and functions in isolation (e.g., `IsekAgent` methods, `Toolkit` functions). The existing `tests/` directory should be expanded.
*   **Integration Tests:**
    *   Test the interaction between the Python `Node` and the `p2p_server.js` sidecar.
    *   Test the full communication flow between two `Isek Node` instances using the local `DefaultRegistry`.
    *   Test the `EtcdRegistry` integration.
*   **End-to-End (E2E) Tests:** The `examples/` scripts serve as E2E tests. A test runner should execute each example script and verify its output to ensure the entire system works as expected.
*   **CI/CD:** The `.github/workflows/ci.yml` should be configured to run all unit and integration tests on every pull request.

## 14. Deployment Plan

1.  **Prerequisites:** The target machine must have Python 3.10+ and Node.js 18+ installed.
2.  **Installation:** The framework is packaged on PyPI. Deployment is done via `pip install isek`.
3.  **Setup:** The `isek setup` command is run to install `npm` dependencies for the P2P server.
4.  **Configuration:** An `.env` file is created with the necessary API keys. For a multi-node deployment, an `etcd` cluster must be running and its address provided to the `Node` configuration.
5.  **Execution:** The user's Python script, which defines and starts the `Isek Node`, is run using `python my_node_app.py`. It is recommended to use a process manager like `systemd` or `supervisor` to run the node as a persistent service.

## 15. Maintenance and Support

*   **Logging:** The framework uses the `loguru` library for logging. Nodes should be configured to write logs to a file for debugging and monitoring.
*   **Dependency Management:** Dependencies are managed in `pyproject.toml`. Regular updates and security audits of dependencies should be performed.
*   **Community Support:** Support is provided through GitHub Issues. A `CONTRIBUTING.md` file guides community contributions.
*   **Monitoring:** For production systems, the FastAPI node server can be integrated with standard monitoring tools like Prometheus and Grafana to track API latency, error rates, and agent activity.

## 16. Future Improvements

The current MVP provides a solid foundation for decentralized agent communication. The following enhancements focus on increasing robustness, performance, and enabling more complex, dynamic agent interactions.

*   **Transition to a Production-Grade Transport Layer with gRPC:**
    *   **Current Status:** The `Isek Node` in `isek/node/node_v2.py` uses FastAPI, providing an HTTP/JSON-based API. This is excellent for compatibility and ease of debugging, but can be a performance bottleneck for high-throughput, low-latency inter-agent communication.
    *   **Proposed Enhancement:** Introduce an `EnhancedIsekNode` that utilizes gRPC for its primary transport. This involves defining a formal service contract with a `.proto` file, which would replace the implicit API defined by FastAPI routes.
    *   **Benefits & Details:**
        *   **Performance:** gRPC uses a binary protocol over HTTP/2, significantly reducing serialization overhead and network latency compared to JSON over HTTP/1.1.
        *   **Strong Typing:** By defining services and messages in a `.proto` file, we enforce a strict, language-agnostic API contract between nodes, reducing runtime errors. The existing Pydantic models can serve as a reference for the Protobuf message definitions.
        *   **Streaming:** gRPC natively supports bidirectional streaming, enabling more complex, real-time interactions like streaming intermediate results or maintaining persistent connections between agents.

*   **Implement Dynamic, Capability-Based A2A Routing:**
    *   **Current Status:** The existing `A2AProtocol` routes messages based on an explicit `receiver_node_id`. An agent must know the specific ID of the node it wants to talk to, which it discovers from the registry. This is a form of static, address-based routing.
    *   **Proposed Enhancement:** Evolve the protocol to support dynamic, capability-based routing. An agent would dispatch a task not to a specific agent, but to a *capability* (e.g., `chat.completion`, `image_generation`, `data_analysis_v2`).
    *   **Benefits & Details:**
        *   **Decoupling & Scalability:** Agents no longer need to know about every other agent. They can simply request a capability, and the network finds a suitable provider. This allows new agents with new skills to join and serve requests without requiring existing agents to be reconfigured.
        *   **Implementation:** This would require enhancing the `AgentCard` to include a list of registered `capabilities`. The `EnhancedA2AProtocol` would then implement a discovery and routing logic. When a task is broadcast, nodes with agents that match the required capability could bid for or claim the task, with the protocol handling the subsequent connection.

*   **Introduce a Robust Task and Event Management System:**
    *   **Current Status:** The framework currently lacks an explicit system for managing the lifecycle of tasks. A task is implicitly handled within a single `agent.run()` call. If a node fails mid-task, its state is lost.
    *   **Proposed Enhancement:** Implement a `Task Store` component within each node.
    *   **Benefits & Details:**
        *   **Reliability & Recovery:** By logging task state changes as events (`task_received`, `tool_started`, `task_completed`), we create an auditable trail. If a node crashes, it could potentially recover and resume tasks by replaying the event log from a persistent store.
        *   **Asynchronous Operations:** A task store is the foundation for handling long-running, asynchronous tasks. An agent could accept a task, return a `task_id` immediately, and the client could poll the task store for status updates.
        *   **Initial Implementation:** The first version would be an in-memory, thread-safe dictionary using `threading.Lock` to manage concurrent access, as suggested. For true scalability and fault tolerance, this would be designed with a pluggable backend, allowing it to be swapped with a persistent, distributed store like Redis or a database in the future.
