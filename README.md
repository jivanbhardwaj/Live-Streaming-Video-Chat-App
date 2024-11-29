# Live-Streaming-Video-Chat-App
# Socket Programming in Python
Socket programming enables communication between two devices over a network. It involves a server that listens for connections and a client that initiates them. This is the backbone of many network-based applications like web browsing, file transfers, and messaging.
What is a Socket?
A socket is an endpoint for two-way communication. It can communicate: 
- Within a process 
- Between processes on the same machine 
- Across networks globally 

Sockets use protocols like TCP (reliable) and UDP (fast) to transmit data.
Types of Sockets in Python
1. TCP Sockets: Reliable, used for tasks like web browsing.
2. UDP Sockets: Fast, used in streaming or gaming.
3. Unix Domain Sockets: For interprocess communication on the same machine.
Components of Socket Programming
1. Server: Listens for client connections, processes requests, and sends responses.
2. Client: Initiates connection, sends requests, and receives responses.
Example: Simple Client-Server Program
Here is my Server Code:

import pickle
import struct
import cv2
import socket

# Set up server details
port = 1254
server_ip = "0.0.0.0"  # Listen on all network interfaces
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.bind(("192.168.56.1", port))  # Bind to the specific IP and port
server_socket.listen(5)  # Allows the server to accept incoming connections

print(f"Server listening on {server_ip}:{port}...")

# Accept client connection
client_socket, client_address = server_socket.accept()
print(f"Connection from {client_address}")

# Set up camera for image capture
camera = cv2.VideoCapture(0)  # 0 is usually the default camera

if not camera.isOpened():
    print("Error: Could not access the camera.")
    exit()

try:
    while True:
        # Capture a frame from the camera
        ret, frame = camera.read()

        if not ret:
            print("Failed to capture image.")
            break

        # Serialize the image using pickle
        img_data = pickle.dumps(frame)

        # Send the size of the image data first
        msg_size = struct.pack("Q", len(img_data))
        client_socket.sendall(msg_size)

        # Send the image data
        client_socket.sendall(img_data)
        print("Image sent to client.")

        # Display the image on the server (optional)
        cv2.imshow("Server Camera", frame)

        # Press 'Enter' to stop the loop
        if cv2.waitKey(1) == 13:
            break
finally:
    # Clean up
    camera.release()
    client_socket.close()
    server_socket.close()
    cv2.destroyAllWindows()
Client Code:

# import library
import cv2
import socket
import pickle
import struct
# create Socket
try:
    skt = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    print("Socket successfully created Now Show ...")
except socket.error as err:
    print("Socket creation failed with error {}".formatat(err))
# main Program Block
port = 1254
server_ip = "192.168.56.1"
skt.connect((server_ip,port))
data = b""
payload_size = struct.calcsize("Q")
try:
    while True:
        while len(data) < payload_size:
            packet = skt.recv(4*1024)
            if not packet: break
            data += packet
        packed_msg_size = data[:payload_size]
        data = data[payload_size:]
        msg_size =  struct.unpack("Q",packed_msg_size)[0]

        while len(data) < msg_size:
            data+= skt.recv(4*1024)
        img_data = data[:msg_size]
        data = data[msg_size:]
        img = pickle.loads(img_data)
        cv2.imshow("Recieving video", img)
        if cv2.waitKey(10) == 13:
            cv2.destroyAllWindows()
            break
    skt.close()
except:
    cv2.destroyAllWindows()
    skt.close()
