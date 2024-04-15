# Simulated Network Traffic Project

This project demonstrates a simple client-server architecture for simulating network traffic. The server listens for incoming connections and records the total bytes received over a period of time, while the client sends simulated network traffic to the server.

## Requirements

- Windows operating system
- Visual Studio (for compiling C++ code)
- Git (optional, for cloning the repository)

## Usage

### Server

1. Clone or download this repository to your local machine.
2. Open the project in Visual Studio.
3. Compile and build the server-side code.
4. Run the server executable.
5. The server will start listening for connections on port 8080.

   ```cpp
   #include <iostream>
   #include <winsock2.h>
   #include <windows.h>

   #pragma comment(lib, "ws2_32.lib")

   const int PORT = 8080;
   const int BUFFER_SIZE = 1024;

   int main() {
       // Initialize Winsock
       WSADATA wsaData;
       int result = WSAStartup(MAKEWORD(2, 2), &wsaData);
       if (result != 0) {
           std::cerr << "WSAStartup failed\n";
           return 1;
       }

       // Create socket
       SOCKET serverSocket = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
       if (serverSocket == INVALID_SOCKET) {
           std::cerr << "Error creating socket\n";
           WSACleanup();
           return 1;
       }

       // Bind socket to address
       sockaddr_in serverAddr;
       serverAddr.sin_family = AF_INET;
       serverAddr.sin_addr.s_addr = INADDR_ANY;
       serverAddr.sin_port = htons(PORT);

       if (bind(serverSocket, (sockaddr*)&serverAddr, sizeof(serverAddr)) == SOCKET_ERROR) {
           std::cerr << "Error binding socket\n";
           closesocket(serverSocket);
           WSACleanup();
           return 1;
       }

       // Listen for connections
       if (listen(serverSocket, SOMAXCONN) == SOCKET_ERROR) {
           std::cerr << "Error listening on socket\n";
           closesocket(serverSocket);
           WSACleanup();
           return 1;
       }
       std::cout << "Server listening on port " << PORT << std::endl;

       // Accept connections
       SOCKET clientSocket = accept(serverSocket, NULL, NULL);
       if (clientSocket == INVALID_SOCKET) {
           std::cerr << "Error accepting connection\n";
           closesocket(serverSocket);
           WSACleanup();
           return 1;
       }

       // Initialize variables
       long totalBytesReceived = 0;
       char buffer[BUFFER_SIZE];

       // Set start time
          DWORD startTime = GetTickCount();
      
          // Receive data loop
          while (true) {
              // Receive data
              int bytesReceived = recv(clientSocket, buffer, BUFFER_SIZE, 0);
              if (bytesReceived == SOCKET_ERROR) {
                  std::cerr << "Error receiving data\n";
                  break;
              }
      
              // Update total bytes received
              totalBytesReceived += bytesReceived;
      
              // Get current time
              DWORD currentTime = GetTickCount();
              // Calculate elapsed time in milliseconds
              DWORD elapsedTime = currentTime - startTime;
      
              // Check if 7 seconds have elapsed
              if (elapsedTime >= 7000) { // 7 seconds = 7000 milliseconds
                  std::cout << "Total bytes received from the beginning: " << totalBytesReceived << std::endl;
                  // Reset start time
                  startTime = GetTickCount();
              }
      
          }
      
          // Close sockets
          closesocket(clientSocket);
          closesocket(serverSocket);
          WSACleanup();
      
          return 0;
      }


### Client

1. Open the project in Visual Studio.
2. Compile and build the client-side code.
3. Run the client executable.
4. The client will connect to the server at `127.0.0.1:8080` and start sending simulated network traffic.
   
    ```cpp
   #include <iostream>
   #include <winsock2.h>
   #include <windows.h>

   #pragma comment(lib, "ws2_32.lib")

   const int PORT = 8080;
   const int BUFFER_SIZE = 1024;

   int main() {
       // Initialize Winsock
       WSADATA wsaData;
       int result = WSAStartup(MAKEWORD(2, 2), &wsaData);
       if (result != 0) {
           std::cerr << "WSAStartup failed\n";
           return 1;
       }
 
       // Create socket
       SOCKET clientSocket = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
       if (clientSocket == INVALID_SOCKET) {
           std::cerr << "Error creating socket\n";
           WSACleanup();
           return 1;
       }

       // Connect to server
       sockaddr_in serverAddr;
       serverAddr.sin_family = AF_INET;
       serverAddr.sin_addr.s_addr = inet_addr(SERVER_IP);
       serverAddr.sin_port = htons(PORT);
   
        if (connect(clientSocket, (sockaddr*)&serverAddr, sizeof(serverAddr)) == SOCKET_ERROR) {
           std::cerr << "Error connecting to server\n";
           closesocket(clientSocket);
           WSACleanup();
            return 1;
      }

       // Generate simulated traffic
       const char* data = "Simulated network traffic";
       while (true) {
           int bytesSent = send(clientSocket, data, strlen(data), 0);
           if (bytesSent == SOCKET_ERROR) {
               std::cerr << "Error sending data\n";
               break;
           }
       }

       // Close socket
       closesocket(clientSocket);
       WSACleanup();

       return 0;
   }


## Additional Notes

- The server-side code (`server.cpp`) records the total bytes received from the client every 7 seconds.
- The client-side code (`client.cpp`) continuously sends simulated network traffic to the server.
- This project uses Winsock2 for socket programming on Windows.
- Feel free to modify the code and experiment with different parameters or functionalities.

