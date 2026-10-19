# QuickChat – PROG5121

## Student Information

**Student Name:** Matimba Hobyani  
**Module:** PROG5121  
**Project:** QuickChat  
**Language:** Java  
**Build Tool:** Maven  
**Java Version:** 17  

## Project Description

QuickChat is a Java-based messaging application developed for the PROG5121 assignment. The application provides user login, message creation, message validation, message ID generation, message hashing, message sending, message storage and unit testing.

## Features

- User registration and login
- Username and password validation
- Message ID generation
- Recipient cellphone number validation
- Message length validation
- Message hash generation
- Send message functionality
- Disregard message functionality
- Store message functionality
- JSON message storage
- JUnit unit testing
- Maven project structure
- Input validation and negative scenarios

## Technologies Used

- Java 17
- Maven
- JUnit 5
- GitHub

---

# Source Code

## Message.java

```java
package za.ac.quickchat;

import java.security.SecureRandom;
import java.util.Locale;

/**
 * Represents one message in QuickChat.
 */
public class Message {
    private static final SecureRandom RANDOM = new SecureRandom();

    private final String messageId;
    private final int messageNumber;
    private final String recipient;
    private final String messageText;
    private final String messageHash;
    private String status;

    public Message(int messageNumber, String recipient, String messageText) {
        this(messageNumber, generateMessageId(), recipient, messageText);
    }

    // Constructor with an ID is useful for predictable unit tests.
    public Message(int messageNumber, String messageId, String recipient, String messageText) {
        this.messageNumber = messageNumber;
        this.messageId = messageId;
        this.recipient = recipient;
        this.messageText = messageText;
        this.messageHash = createMessageHash();
        this.status = "Pending";
    }

    public static String generateMessageId() {
        long value = 1_000_000_000L
                + (long) (RANDOM.nextDouble() * 9_000_000_000L);

        return String.valueOf(value);
    }

    public boolean checkMessageID() {
        return messageId != null && messageId.matches("\\d{1,10}");
    }

    public String checkRecipientCell() {
        // South African international format: +27 followed by nine digits.
        if (recipient != null && recipient.matches("\\+27\\d{9}")) {
            return "Cell phone number successfully captured.";
        }

        return "Cell phone number is incorrectly formatted or does not contain an international code. Please correct the number and try again.";
    }

    public String validateMessageLength() {
        if (messageText == null) {
            return "Message ready to send.";
        }

        if (messageText.length() <= 250) {
            return "Message ready to send.";
        }

        return "Message exceeds 250 characters by "
                + (messageText.length() - 250)
                + "; please reduce the size.";
    }

    public String createMessageHash() {
        String firstTwoDigits =
                messageId != null && messageId.length() >= 2
                        ? messageId.substring(0, 2)
                        : "00";

        String firstWord = "";
        String lastWord = "";

        if (messageText != null && !messageText.trim().isEmpty()) {
            String[] words = messageText.trim().split("\\s+");

            firstWord = words[0].replaceAll("[^A-Za-z0-9]", "");
            lastWord = words[words.length - 1]
                    .replaceAll("[^A-Za-z0-9]", "");
        }

        return (firstTwoDigits + ":" + messageNumber + ":"
                + firstWord + lastWord).toUpperCase(Locale.ROOT);
    }

    public String sentMessage(int choice) {
        switch (choice) {
            case 1:
                status = "Sent";
                return "Message successfully sent.";

            case 2:
                status = "Disregarded";
                return "Press 0 to delete the message.";

            case 3:
                status = "Stored";
                return "Message successfully stored.";

            default:
                return "Invalid option. Choose 1, 2, or 3.";
        }
    }

    public String printMessages() {
        return "Message ID: " + messageId
                + System.lineSeparator()
                + "Message Hash: " + messageHash
                + System.lineSeparator()
                + "Recipient: " + recipient
                + System.lineSeparator()
                + "Message: " + messageText;
    }

    public int returnTotalMessagess() {
        return "Sent".equals(status) ? 1 : 0;
    }

    public String toJson() {
        return "{"
                + "\"messageId\":\"" + escapeJson(messageId) + "\","
                + "\"messageNumber\":" + messageNumber + ","
                + "\"messageHash\":\"" + escapeJson(messageHash) + "\","
                + "\"recipient\":\"" + escapeJson(recipient) + "\","
                + "\"message\":\"" + escapeJson(messageText) + "\","
                + "\"status\":\"" + escapeJson(status) + "\""
                + "}";
    }

    private static String escapeJson(String value) {
        if (value == null) {
            return "";
        }

        return value.replace("\\", "\\\\")
                .replace("\"", "\\\"")
                .replace("\n", "\\n")
                .replace("\r", "\\r")
                .replace("\t", "\\t");
    }

    public String getMessageId() {
        return messageId;
    }

    public int getMessageNumber() {
        return messageNumber;
    }

    public String getRecipient() {
        return recipient;
    }

    public String getMessageText() {
        return messageText;
    }

    public String getMessageHash() {
        return messageHash;
    }

    public String getStatus() {
        return status;
    }
}



package za.ac.quickchat;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.StandardOpenOption;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

/**
 * Console entry point for the Part 2 QuickChat application.
 */
public class QuickChat {

    private static final Scanner INPUT = new Scanner(System.in);
    private static final List<Message> MESSAGES = new ArrayList<>();

    public static void main(String[] args) {

        System.out.println("=== QuickChat Login ===");

        System.out.print("Create a username for this session: ");
        String registeredUsername = INPUT.nextLine().trim();

        System.out.print("Create a password for this session: ");
        String registeredPassword = INPUT.nextLine();

        System.out.println("\nPlease log in.");

        boolean loggedIn = false;

        for (int attempt = 1; attempt <= 3; attempt++) {

            System.out.print("Username: ");
            String username = INPUT.nextLine().trim();

            System.out.print("Password: ");
            String password = INPUT.nextLine();

            if (username.equals(registeredUsername)
                    && password.equals(registeredPassword)) {

                loggedIn = true;

                System.out.println("Login successful.");

                break;
            }

            System.out.println(
                    "Username or password incorrect, please try again.");
        }

        if (!loggedIn) {

            System.out.println(
                    "Login failed. QuickChat will close.");

            return;
        }

        System.out.println("Welcome to QuickChat.");

        boolean running = true;

        while (running) {

            displayMenu();

            int option = readInt("Choose an option: ");

            switch (option) {

                case 1:
                    sendMessages();
                    break;

                case 2:
                    System.out.println("Coming Soon.");
                    break;

                case 3:
                    running = false;
                    System.out.println(
                            "Thank you for using QuickChat.");
                    break;

                default:
                    System.out.println(
                            "Invalid option. Please choose 1, 2, or 3.");
            }
        }
    }

    private static void displayMenu() {

        System.out.println("\n1) Send Messages");
        System.out.println("2) Show recently sent messages");
        System.out.println("3) Quit");
    }

    private static void sendMessages() {

        int numberOfMessages =
                readInt("How many messages would you like to enter? ");

        while (numberOfMessages < 1) {

            System.out.println(
                    "Please enter a number greater than zero.");

            numberOfMessages =
                    readInt("How many messages would you like to enter? ");
        }

        int sentCount = 0;

        for (int i = 1; i <= numberOfMessages; i++) {

            System.out.println(
                    "\n--- Message "
                    + i
                    + " of "
                    + numberOfMessages
                    + " ---");

            String recipient;

            while (true) {

                System.out.print(
                        "Recipient number (format +27XXXXXXXXX): ");

                recipient = INPUT.nextLine().trim();

                Message check =
                        new Message(
                                i,
                                "0000000000",
                                recipient,
                                "");

                String validation =
                        check.checkRecipientCell();

                if (validation.equals(
                        "Cell phone number successfully captured.")) {

                    System.out.println(validation);

                    break;
                }

                System.out.println(validation);
            }

            System.out.print(
                    "Enter your message (maximum 250 characters): ");

            String text = INPUT.nextLine();

            Message message =
                    new Message(i, recipient, text);

            String lengthResult =
                    message.validateMessageLength();

            while (!lengthResult.equals(
                    "Message ready to send.")) {

                System.out.println(lengthResult);

                System.out.print(
                        "Enter a shorter message "
                        + "(maximum 250 characters): ");

                text = INPUT.nextLine();

                message =
                        new Message(i, recipient, text);

                lengthResult =
                        message.validateMessageLength();
            }

            System.out.println(lengthResult);

            System.out.println(
                    "Message ID generated: "
                    + message.getMessageId());

            System.out.println(
                    "Message Hash: "
                    + message.getMessageHash());

            System.out.println("1) Send Message");
            System.out.println("2) Disregard Message");
            System.out.println(
                    "3) Store Message to send later");

            int action;

            while (true) {

                action = readInt("Choose an action: ");

                if (action >= 1 && action <= 3) {
                    break;
                }

                System.out.println(
                        "Invalid option. Please choose 1, 2, or 3.");
            }

            String actionResult =
                    message.sentMessage(action);

            System.out.println(actionResult);

            if (action == 1) {

                sentCount++;

            } else if (action == 3) {

                try {

                    storeMessage(message);

                } catch (IOException exception) {

                    System.out.println(
                            "Could not store the message: "
                            + exception.getMessage());
                }
            }

            MESSAGES.add(message);

            System.out.println(
                    "\n" + message.printMessages());
        }

        System.out.println(
                "\nTotal messages sent: " + sentCount);
    }

    private static void storeMessage(Message message)
            throws IOException {

        Path file = Path.of("stored_messages.json");

        String existing =
                Files.exists(file)
                        ? Files.readString(
                                file,
                                StandardCharsets.UTF_8).trim()
                        : "";

        String object = message.toJson();

        String updated;

        if (existing.isEmpty()
                || existing.equals("[]")) {

            updated =
                    "[\n  "
                    + object
                    + "\n]";

        } else {

            String content =
                    existing.substring(
                            0,
                            existing.length() - 1).trim();

            updated =
                    content
                    + ",\n  "
                    + object
                    + "\n]";
        }

        Files.writeString(
                file,
                updated,
                StandardCharsets.UTF_8,
                StandardOpenOption.CREATE,
                StandardOpenOption.TRUNCATE_EXISTING);
    }

    private static int readInt(String prompt) {

        while (true) {

            System.out.print(prompt);

            String value =
                    INPUT.nextLine().trim();

            try {

                return Integer.parseInt(value);

            } catch (NumberFormatException exception) {

                System.out.println(
                        "Please enter a valid number.");
            }
        }
    }
}




package za.ac.quickchat;

import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class MessageTest {

    @Test
    void messageIdMustNotExceedTenDigits() {

        Message valid =
                new Message(
                        1,
                        "1234567890",
                        "+27718693002",
                        "Hi Mike, can you join us for dinner tonight?");

        Message invalid =
                new Message(
                        1,
                        "12345678901",
                        "+27718693002",
                        "Hello");

        assertTrue(valid.checkMessageID());
        assertFalse(invalid.checkMessageID());
    }

    @Test
    void recipientNumberMustUseSouthAfricanInternationalFormat() {

        Message valid =
                new Message(
                        1,
                        "1234567890",
                        "+27718693002",
                        "Hello");

        Message invalid =
                new Message(
                        1,
                        "1234567890",
                        "08575975889",
                        "Hello");

        assertEquals(
                "Cell phone number successfully captured.",
                valid.checkRecipientCell());

        assertEquals(
                "Cell phone number is incorrectly formatted or does not contain an international code. Please correct the number and try again.",
                invalid.checkRecipientCell());
    }

    @Test
    void messageMustNotExceed250Characters() {

        Message valid =
                new Message(
                        1,
                        "0012345678",
                        "+27718693002",
                        "Hi Mike");

        Message invalid =
                new Message(
                        1,
                        "0012345678",
                        "+27718693002",
                        "a".repeat(253));

        assertEquals(
                "Message ready to send.",
                valid.validateMessageLength());

        assertEquals(
                "Message exceeds 250 characters by 3; please reduce the size.",
                invalid.validateMessageLength());
    }

    @Test
    void messageHashUsesFirstTwoIdDigitsMessageNumberAndFirstLastWords() {

        Message message =
                new Message(
                        0,
                        "0012345678",
                        "+27718693002",
                        "Hi Mike, can you join us for dinner tonight?");

        assertEquals(
                "00:0:HITONIGHT",
                message.createMessageHash());
    }

    @Test
    void messageActionReturnsRequiredMessages() {

        Message sent =
                new Message(
                        1,
                        "1234567890",
                        "+27718693002",
                        "Hello");

        Message disregarded =
                new Message(
                        2,
                        "1234567891",
                        "+27718693002",
                        "Hello");

        Message stored =
                new Message(
                        3,
                        "1234567892",
                        "+27718693002",
                        "Hello");

        assertEquals(
                "Message successfully sent.",
                sent.sentMessage(1));

        assertEquals(
                "Press 0 to delete the message.",
                disregarded.sentMessage(2));

        assertEquals(
                "Message successfully stored.",
                stored.sentMessage(3));
    }

    @Test
    void generatedMessageIdHasTenDigits() {

        String id =
                Message.generateMessageId();

        assertTrue(
                id.matches("\\d{10}"));
    }
}


<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>za.ac.quickchat</groupId>

    <artifactId>QuickChat</artifactId>

    <version>1.0-SNAPSHOT</version>

    <properties>

        <maven.compiler.release>17</maven.compiler.release>

        <project.build.sourceEncoding>
            UTF-8
        </project.build.sourceEncoding>

        <junit.version>5.10.2</junit.version>

    </properties>

    <dependencies>

        <dependency>

            <groupId>org.junit.jupiter</groupId>

            <artifactId>junit-jupiter</artifactId>

            <version>${junit.version}</version>

            <scope>test</scope>

        </dependency>

    </dependencies>

    <build>

        <plugins>

            <plugin>

                <groupId>org.apache.maven.plugins</groupId>

                <artifactId>maven-surefire-plugin</artifactId>

                <version>3.2.5</version>

            </plugin>

        </plugins>

    </build>

</project>


**README**


**Repository**
```text
QuickChat/
├── pom.xml
├── README.md
└── src/
    ├── main/
    │   └── java/
    │       └── za/
    │           └── ac/
    │               └── quickchat/
    │                   ├── Message.java
    │                   └── QuickChat.java
    └── test/
        └── java/
            └── za/
                └── ac/
                    └── quickchat/
                        └── MessageTest.java
