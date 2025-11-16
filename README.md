# SimpleTorrent

A peer-to-peer file sharing application built in Java that implements a simplified BitTorrent-like protocol for distributed file transfers across a network of nodes.

## Overview

SimpleTorrent is a distributed file-sharing system where multiple nodes can connect to each other and share MP3 files. The application uses a decentralized architecture where each node acts as both a client and a server, allowing efficient parallel downloads from multiple sources simultaneously.

## Features

- **Peer-to-Peer Architecture**: Each node can act as both client and server
- **Parallel Downloads**: Files are downloaded in blocks from multiple sources concurrently
- **File Search**: Search for files across connected nodes by keyword
- **Multi-threaded**: Efficient concurrent processing of requests and downloads
- **GUI Interface**: User-friendly Swing-based graphical interface
- **Hash-based Identification**: Files are identified using SHA-256 hashing
- **Block-based Transfer**: Files are split into 10KB blocks for efficient transfer

## Architecture

### Core Components

#### 1. **Node**
The main node class that manages all peer-to-peer operations:
- Runs a server socket to accept incoming connections
- Maintains connections to other nodes through `NodeAgent` instances
- Manages file search operations across the network
- Coordinates download tasks through `DownloadTaskManager`
- Processes file block requests from other nodes using a thread pool

#### 2. **NodeAgent**
Handles communication between nodes:
- Manages the socket connection between two nodes
- Sends and receives serialized objects (messages, requests, responses)
- Routes different message types to appropriate handlers
- Each node can have multiple `NodeAgent` instances for different connections

#### 3. **DownloadTaskManager**
Orchestrates the download process for a single file:
- Creates a list of block requests for the entire file
- Manages multiple requester threads (one per available source)
- Uses a `CountDownLatch` to wait for all blocks to complete
- Tracks statistics about which nodes provided which blocks
- Reconstructs the file once all blocks are received

#### 4. **IscTorrentGUI**
The graphical user interface:
- Allows users to search for files by keyword
- Displays search results with file hash and number of sources
- Enables connection to other nodes
- Shows download completion statistics
- Runs on port 8080 + node ID

### Message Types

The application uses several serializable message classes for communication:

1. **WordSearchMessage**: Request to search for files containing a keyword
2. **FileSearchResult**: Information about a file (hash, size, name, source location)
3. **FileBlockRequestMessage**: Request for a specific block of a file
4. **FileBlockAnswerMessage**: Response containing the requested block data
5. **NewConnectionRequest**: Initial handshake when nodes connect

### Utility Classes

#### FileUtils
Provides essential file operations:
- `hashValue()`: Calculates SHA-256 hash of files
- `createFileBlockRequestList()`: Divides file into block requests
- `readFileBlock()`: Reads a specific block from a file
- `createFile()`: Reconstructs a file from received blocks
- `getFilesByWord()`: Searches for files matching a keyword

#### CountDownLatch
Custom synchronization primitive that allows threads to wait until a set of operations complete.

#### NodeAgentTask
Generic wrapper class that pairs a task with the `NodeAgent` that submitted it.

## How It Works

### 1. Node Connection
- Each node starts a server on port 8080 + node ID
- Users can connect to other nodes via the GUI by providing address and port
- Nodes exchange `NewConnectionRequest` messages to establish the connection

### 2. File Search
- User enters a search keyword in the GUI
- The node sends `WordSearchMessage` to all connected nodes
- Each node searches its local folder and returns matching `FileSearchResult` objects
- Results are aggregated and displayed, grouped by file hash

### 3. File Download
- User selects a file to download from search results
- A `DownloadTaskManager` is created with all nodes that have the file
- The file is divided into 10KB blocks (configurable via `BLOCK_SIZE`)
- Multiple requester threads request blocks in parallel from available nodes
- Each node processes requests using a thread pool
- Blocks are reassembled in order once all are received
- Statistics are displayed showing contribution from each source

### 4. Serving Files
- Each node continuously listens for incoming requests
- File block requests are queued and processed by a thread pool
- The node reads the requested block and sends a `FileBlockAnswerMessage`

## Project Structure

```
SimpleTorrent/
├── Main.java                      # Application entry point
├── Node.java                      # Core node implementation
├── NodeAgent.java                 # Inter-node communication handler
├── NodeAgentTask.java             # Task wrapper with associated agent
├── DownloadTaskManager.java       # Download coordination
├── IscTorrentGUI.java            # Graphical user interface
├── FileUtils.java                 # File operations and utilities
├── CountDownLatch.java            # Synchronization primitive
├── FileSearchResult.java          # File search result data
├── FileBlockRequestMessage.java   # Block request message
├── FileBlockAnswerMessage.java    # Block response message
├── WordSearchMessage.java         # Search query message
├── NewConnectionRequest.java      # Connection handshake message
└── folders/                       # Storage directories
    ├── dl1/                       # Node 1's files
    ├── dl2/                       # Node 2's files
    ├── dl3/                       # Node 3's files
    └── dl4/                       # Node 4's files
```

## Usage

### Running the Application

To start a node, run the `Main` class with a node ID as argument:

```bash
java Main 1
```

This will start a node on port 8081 (8080 + 1) using the folder `folders/dl1`.

### Starting Multiple Nodes

To simulate a P2P network, start multiple instances with different IDs:

```bash
java Main 1
java Main 2
java Main 3
java Main 4
```

### Connecting Nodes

1. Click "Ligar a Nó" (Connect to Node) button
2. Enter the address (default: localhost) and port of another node
3. Click OK to establish connection

### Searching for Files

1. Enter a search keyword in the text field
2. Click "Procurar" (Search) button
3. Results will appear showing: filename [hash] (number of sources)

### Downloading Files

1. Select one or more files from the search results
2. Click "Descarregar" (Download) button
3. A completion dialog will show download statistics

## Technical Details

### Threading Model
- Server socket runs in a separate thread per node
- Each `NodeAgent` connection runs in its own thread
- File block requests are processed by a fixed thread pool (size 5)
- Each `DownloadTaskManager` spawns multiple requester threads

### Synchronization
- Synchronized methods protect shared data structures
- `wait()`/`notifyAll()` used for thread coordination
- Custom `CountDownLatch` ensures all blocks are received before file reconstruction

### File Identification
- Files are identified by SHA-256 hash (truncated to int)
- Same file from different sources has the same hash
- Allows downloading from multiple sources simultaneously

### Block Size
- Default block size: 10KB (10,240 bytes)
- Configurable via `FileUtils.BLOCK_SIZE`
- Files are divided into full blocks + remainder

## Limitations

- Only supports MP3 files (`.mp3` extension)
- Uses localhost by default
- Fixed thread pool size
- Hash collision possible (using truncated SHA-256)
- No encryption or authentication
- No file integrity verification after download

## Requirements

- Java 8 or higher
- Swing GUI library (included in standard Java)
- Network connectivity between nodes

## Future Enhancements

Potential improvements could include:
- Support for more file types
- DHT (Distributed Hash Table) for node discovery
- Resume capability for interrupted downloads
- Bandwidth throttling
- File integrity verification (checksums)
- Encryption for secure transfers
- Support for NAT traversal
- GUI improvements and better statistics

## License

This is an educational project for learning distributed systems and P2P networking concepts.
