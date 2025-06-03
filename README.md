# CHAT-APPLICATION
#Server
import java.io.*;
import java.net.*;
import java.util.*;

public class Server {
    private static final int PORT = 12345;
    private static Set<ClientHandler> clients = Collections.synchronizedSet(new HashSet<>());

    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(PORT);
        System.out.println("Server started on port " + PORT);

        while (true) {
            Socket socket = serverSocket.accept();
            ClientHandler client = new ClientHandler(socket, clients);
            clients.add(client);
            new Thread(client).start();
        }
    }
}


#ClientHandler
#import java.io.*;
import java.net.*;
import java.util.*;

public class ClientHandler implements Runnable {
    private Socket socket;
    private ObjectInputStream in;
    private ObjectOutputStream out;
    private Set<ClientHandler> clients;

    public ClientHandler(Socket socket, Set<ClientHandler> clients) {
        this.socket = socket;
        this.clients = clients;
        try {
            out = new ObjectOutputStream(socket.getOutputStream());
            in = new ObjectInputStream(socket.getInputStream());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void run() {
        try {
            Object obj;
            while ((obj = in.readObject()) != null) {
                broadcast(obj);
            }
        } catch (Exception e) {
            System.out.println("Client disconnected.");
        } finally {
            try {
                clients.remove(this);
                socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    private void broadcast(Object message) {
        synchronized (clients) {
            for (ClientHandler client : clients) {
                try {
                    client.out.writeObject(message);
                    client.out.flush();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

#Client
public class Client {
    public static void main(String[] args) {
        new ChatUI("User123"); // Replace with login ID
    }
}

#ChatUI
import javafx.application.Application;
import javafx.scene.Scene;
import javafx.scene.control.*;
import javafx.scene.layout.*;
import javafx.stage.Stage;

public class ChatUI extends Application {
    private String username;

    public ChatUI(String username) {
        this.username = username;
        launch();
    }

    @Override
    public void start(Stage primaryStage) {
        BorderPane root = new BorderPane();
        VBox chatBox = new VBox();
        ScrollPane scrollPane = new ScrollPane(chatBox);
        TextField input = new TextField();
        Button send = new Button("Send");

        HBox inputBox = new HBox(input, send);
        root.setCenter(scrollPane);
        root.setBottom(inputBox);

        // Dark/Light mode toggle
        ToggleButton themeToggle = new ToggleButton("Dark Mode");
        themeToggle.setOnAction(e -> {
            if (themeToggle.isSelected()) {
                root.setStyle("-fx-background-color: #333333; -fx-text-fill: white;");
            } else {
                root.setStyle("-fx-background-color: white; -fx-text-fill: black;");
            }
        });
        root.setTop(themeToggle);

        Scene scene = new Scene(root, 400, 600);
        primaryStage.setTitle("Next Gen Chat - " + username);
        primaryStage.setScene(scene);
        primaryStage.show();
    }
}

#Authenticator
import java.util.Map;

public class Authenticator {
    private static Map<String, String> users = Map.of("admin", "1234", "user1", "pass");

    public static boolean authenticate(String username, String password) {
        return users.containsKey(username) && users.get(username).equals(password);
    }
}
