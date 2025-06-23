# A-CLIENT-SERVER-CHAT-APPLICATION-USING-JAVA-SOCKETS-AND-MULTITHREADING-TO-HANDLE-MULTIPLE-USERS.
FUNCTIONAL CHAT APPLICATION WITH A SERVER AND MULTIPLE CLIENTS COMMUNICATING IN REAL TIME.
 1. Server Code (Server.java)
import java.io.*;
import java.net.*;
import java.util.*;

public class Server {
    private static final int PORT = 1234;
    private static Set<ClientHandler> clientHandlers = new HashSet<>();

    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(PORT);
        System.out.println("Server started. Waiting for clients...");

        while (true) {
            Socket socket = serverSocket.accept();
            System.out.println("New client connected: " + socket);

            ClientHandler handler = new ClientHandler(socket);
            clientHandlers.add(handler);
            new Thread(handler).start();
        }
    }

    public static void broadcast(String message, ClientHandler sender) {
        for (ClientHandler client : clientHandlers) {
            if (client != sender) {
                client.sendMessage(message);
            }
        }
    }

    public static void removeClient(ClientHandler client) {
        clientHandlers.remove(client);
    }
}

class ClientHandler implements Runnable {
    private Socket socket;
    private PrintWriter out;
    private BufferedReader in;
    private String name;

    public ClientHandler(Socket socket) {
        this.socket = socket;
    }

    public void run() {
        try {
            in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            out = new PrintWriter(socket.getOutputStream(), true);

            out.println("Enter your name:");
            name = in.readLine();
            System.out.println(name + " joined.");
            Server.broadcast(name + " joined the chat!", this);

            String message;
            while ((message = in.readLine()) != null) {
                if (message.equalsIgnoreCase("exit")) {
                    break;
                }
                Server.broadcast(name + ": " + message, this);
            }
        } catch (IOException e) {
            System.out.println("Error: " + e.getMessage());
        } finally {
            try {
                Server.removeClient(this);
                Server.broadcast(name + " left the chat.", this);
                socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    public void sendMessage(String message) {
        out.println(message);
    }
}
2. Client Code (Client.java)
import java.io.*;
import java.net.*;

public class Client {
    private static final String SERVER_IP = "localhost"; // or server IP
    private static final int SERVER_PORT = 1234;

    public static void main(String[] args) {
        try {
            Socket socket = new Socket(SERVER_IP, SERVER_PORT);
            System.out.println("Connected to server");

            new Thread(new ReceiveHandler(socket)).start();
            new Thread(new SendHandler(socket)).start();

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

class SendHandler implements Runnable {
    private Socket socket;

    public SendHandler(Socket socket) {
        this.socket = socket;
    }

    public void run() {
        try {
            PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
            BufferedReader console = new BufferedReader(new InputStreamReader(System.in));
            String message;
            while ((message = console.readLine()) != null) {
                out.println(message);
                if (message.equalsIgnoreCase("exit")) {
                    socket.close();
                    break;
                }
            }
        } catch (IOException e) {
            System.out.println("Disconnected from server.");
        }
    }
}

class ReceiveHandler implements Runnable {
    private Socket socket;

    public ReceiveHandler(Socket socket) {
        this.socket = socket;
    }

    public void run() {
        try {
            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            String message;
            while ((message = in.readLine()) != null) {
                System.out.println(message);
            }
        } catch (IOException e) {
            System.out.println("Connection closed.");
        }
    }
}

