# Cryptographic-protocol
Cryptographic data transfer protocol.

The protocol is presentation layer related and serves as an infrastructure for TCP-based application layer protocols, providing data encryption/decryption and serializations.

## Core features
* Hybrid encryption, using both RSA and AES algorithms.
* Digital signatures.
* Supports the transfer of raw bytes, plain text, files and serialized objects.
* Data compression.
* Non-blocking socket connections.
* Simple API.

## How to use
### Establish connection
#### Keys
Generates private and public keys pair (See Public-key cryptography)
```Java
KeyPair keyPair = Handshake.generateKeyPair(600);
```
Generates a secret key (Symmetric cryptography) which is used to encrypt the transferred data
```Java
SecretKey key = Handshake.generateSecretKey(128);
```
#### Handshake
* `SecretKey serverHandshake(@NotNull Wrapper socket, @NotNull KeyPair keys)`

  Establishes a connection from the server's edge. Receives the socket wrapper of the client and the server's keys pair. Returns the client's secret key.
  
* `boolean clientHandshake(@NotNull Wrapper socket, @NotNull SecretKey key)`

  Establishes a connection from the client's edge. Receives the socket wraper of the client's secret key. Returns true if the connection was successfuly established.
### Server
```Java
public class Server {
    public Server() throws IOException, RSAException, InvalidKeyException, SessionInitException, DataFormatException, VerificationException {
        int IP = 64512;
        ServerSocket server = new ServerSocket();
        // Creates private and public keys
        KeyPair key = Handshake.generateKeyPair(600);
        if (key != null){
            Socket socket = server.accept();
            SocketWrapper wrapper = new SocketWrapper(socket);
            // performs handshake with the client and exchange keys
            SecretKey secretKey = Handshake.serverHandshake(wrapper, key);
            Session session = new Session(wrapper, secretKey);
            String message = session.receiveText();
            System.out.println(message);
        }
    }
}
```
### Client
```Java
public class Client {
    public Client(String ip, int port) throws IOException, InvalidKeyException, RSAException, InvalidAESKeySizeException, SessionInitException {
        Socket socket = new Socket(ip, port);
        SocketWrapper wrapper = new SocketWrapper(socket);
        SecretKey secretKey = Handshake.generateSecretKey(128);
        if (secretKey != null) {
            boolean connEstablished = Handshake.clientHandshake(wrapper, secretKey);
            if (connEstablished) {
                Session session = new Session(wrapper, secretKey);
                session.sendText("new message from client");
            }
        }
    }
}
```
### Session
#### Raw bytes and text
* `void send(@NotNull byte[] data)`
* `byte[] receive()`
* `void sendText(@NotNull String text)`
* `String receiveText()`
#### Files
* `void sendFile(@NotNull String filepath)`

  Sends a file in one chunk. Receives file's path.
  
 * `void sendFile(@NotNull String filepath, int chunk)`
 
  Sends a file in chunks. Receives file's path and chunk's size.
  
 * `FileInfo receiveFile()`
 
  Receives a file. Returns FileInfo object.
  
 * `FileInfo receiveFile(@NotNull String savePath, String name)` 
 
  Receives a file. Returns FileInfo object. Parameters are the file's saving destination path and file's name.
  #### FileInfo
  FileInfo object contains information about the received file:
```Java
FileInfo fileInfo = session.receiveFile();

// returns file's name
String filename = fileInfo.getFilename();
// returns file's size
int size = fileInfo.getSize();
// returns file's data
byte[] data = fileInfo.getData();
```
#### Objects
Sending and receiving objects through the session. Objects must be serializable.
```Java
public class Student implements Serializable {
    public String name;
    public int age;

    Student(String name, int age){
        this.name = name;
        this.age = age;
    }
}
```
Send and receive:
```Java
Student student = new Student("student name", 19);
// send object
session.sendObject()
// receive object
Student student = (Student)session.receiveObject();
```

#### Other
* `void setCompressionMode(boolean mode)`

  Sets on/off the compression option.
  
### Exceptions
* WrapperException
* InvalidAESKeySizeException
* RSAException
* SessionInitException
* VerificationException
* NonSerializableClass
