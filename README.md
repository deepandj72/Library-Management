I) DOCKER



1.Form Validation code in react

structure:

JavaFormValidationApp/
├── src/
│   ├── FormServer.java
├── Dockerfile
└── README.md



1)BMIServer.java

import java.io.*;
import java.net.*;
import java.util.regex.*;

public class FormServer {
    public static void main(String[] args) throws IOException {
        int port = 9091;
        ServerSocket serverSocket = new ServerSocket(port);
        System.out.println("Form Validation Server running on http://localhost:" + port);

        while (true) {
            Socket socket = serverSocket.accept();
            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            BufferedWriter out = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));

            String requestLine;
            String query = "";
            while ((requestLine = in.readLine()) != null && !requestLine.isEmpty()) {
                if (requestLine.startsWith("GET")) {
                    int qIndex = requestLine.indexOf("?");
                    if (qIndex != -1) {
                        int end = requestLine.indexOf(" ", qIndex);
                        query = requestLine.substring(qIndex + 1, end);
                    }
                }
            }

            String name = "", email = "", msg = "";
            if (!query.isEmpty()) {
                String[] params = query.split("&");
                for (String p : params) {
                    String[] kv = p.split("=");
                    if (kv[0].equals("name")) name = kv[1].replace("+", " ");
                    if (kv[0].equals("email")) email = kv[1];
                }

                Pattern emailPattern = Pattern.compile("^[\\w.-]+@[\\w.-]+\\.\\w+$");
                if (name.isEmpty() || email.isEmpty()) msg = "All fields are required!";
                else if (!emailPattern.matcher(email).matches()) msg = "Invalid email address!";
                else msg = "Form submitted successfully!";
            }

            String html = """
                <html>
                <head><title>Form Validation</title></head>
                <body style="font-family:Arial;text-align:center;">
                    <h1>Form Validation</h1>
                    <form method="GET" action="/">
                        <label>Name:</label><input type="text" name="name"><br><br>
                        <label>Email:</label><input type="text" name="email"><br><br>
                        <input type="submit" value="Submit">
                    </form>
            """;

            if (!msg.isEmpty()) html += "<h2>" + msg + "</h2>";

            html += "</body></html>";

            out.write("HTTP/1.1 200 OK\r\n");
            out.write("Content-Type: text/html\r\n");
            out.write("Content-Length: " + html.length() + "\r\n\r\n");
            out.write(html);
            out.flush();
            socket.close();
        }
    }
}


2)Dockerfile

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY build/FormServer.class /app/
EXPOSE 9091
CMD ["java", "FormServer"]



commands to run

javac src\FormServer.java -d build
docker build -t java-form-server .
docker run -p 9091:9091 java-form-server



2.BMI calculator or currency converter in Java

structure:

JavaBMIApp/
├── out/
│  
├── src/
│   ├── BMIServer.java
 
├── Dockerfile
└── README.md


and codes are

1)BMIServer.java

import java.io.*;
import java.net.*;

public class BMIServer {
    public static void main(String[] args) throws IOException {
        int port = 9090;
        ServerSocket serverSocket = new ServerSocket(port);
        System.out.println("BMI Server running on http://localhost:" + port);

        while (true) {
            Socket clientSocket = serverSocket.accept();
            BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
            BufferedWriter out = new BufferedWriter(new OutputStreamWriter(clientSocket.getOutputStream()));

            String requestLine;
            String query = "";
            while ((requestLine = in.readLine()) != null && !requestLine.isEmpty()) {
                if (requestLine.startsWith("GET")) {
                    int qIndex = requestLine.indexOf("?");
                    if (qIndex != -1) {
                        int end = requestLine.indexOf(" ", qIndex);
                        query = requestLine.substring(qIndex + 1, end);
                    }
                }
            }

            double bmi = 0;
            String result = "";

            if (!query.isEmpty()) {
                String[] params = query.split("&");
                double w = 0, h = 0;
                for (String p : params) {
                    String[] kv = p.split("=");
                    if (kv[0].equals("w")) w = Double.parseDouble(kv[1]);
                    if (kv[0].equals("h")) h = Double.parseDouble(kv[1]);
                }
                if (h > 0) bmi = w / (h * h);
                if (bmi < 18.5) result = "Underweight";
                else if (bmi < 25) result = "Normal";
                else if (bmi < 30) result = "Overweight";
                else result = "Obese";
            }

            String html = """
                <html>
                <head><title>BMI Calculator</title></head>
                <body style="font-family:Arial;text-align:center;">
                    <h1>BMI Calculator</h1>
                    <form method="GET" action="/">
                        <label>Weight (kg):</label><input type="text" name="w"><br><br>
                        <label>Height (m):</label><input type="text" name="h"><br><br>
                        <input type="submit" value="Calculate">
                    </form>
            """;

            if (bmi > 0)
                html += "<h2>Your BMI: " + String.format("%.2f", bmi) + " (" + result + ")</h2>";

            html += "</body></html>";

            out.write("HTTP/1.1 200 OK\r\n");
            out.write("Content-Type: text/html\r\n");
            out.write("Content-Length: " + html.length() + "\r\n");
            out.write("\r\n");
            out.write(html);
            out.flush();

            clientSocket.close();
        }
    }
}


2)Dockerfile

# Use lightweight Java Runtime (JRE only)
FROM eclipse-temurin:21-jre-alpine

# Set working directory
WORKDIR /app

# Copy precompiled class file
COPY build/BMIServer.class /app/

# Expose port 9090
EXPOSE 9090

# Run the server
CMD ["java", "BMIServer"]


commands to run

1) javac src\BMIServer.java -d build
(O/P) E:\Python\JavaBMIApp\build\BMIServer.class
2) docker build -t java-bmi-server .
3)docker run -p 9090:9090 java-bmi-server
(O/P)LocalHost



3.Simple CRUD application (locally or globally)

structure:

JavaCRUDApp/
├── src/
│   ├── CRUDServer.java
├── Dockerfile
└── README.md




1)BMIServer.java

import java.io.*;
import java.net.*;
import java.util.*;

public class CRUDServer {
    private static Map<Integer, String> data = new HashMap<>();
    private static int idCounter = 1;

    public static void main(String[] args) throws IOException {
        int port = 9092;
        ServerSocket serverSocket = new ServerSocket(port);
        System.out.println("CRUD Server running on http://localhost:" + port);

        while (true) {
            Socket socket = serverSocket.accept();
            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            BufferedWriter out = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));

            String requestLine;
            String query = "";
            while ((requestLine = in.readLine()) != null && !requestLine.isEmpty()) {
                if (requestLine.startsWith("GET")) {
                    int qIndex = requestLine.indexOf("?");
                    if (qIndex != -1) {
                        int end = requestLine.indexOf(" ", qIndex);
                        query = requestLine.substring(qIndex + 1, end);
                    }
                }
            }

            String msg = "";
            if (!query.isEmpty()) {
                String[] params = query.split("&");
                String action = "", name = "";
                int id = 0;

                for (String p : params) {
                    String[] kv = p.split("=");
                    if (kv[0].equals("action")) action = kv[1];
                    if (kv[0].equals("name")) name = kv[1].replace("+", " ");
                    if (kv[0].equals("id")) id = Integer.parseInt(kv[1]);
                }

                switch (action) {
                    case "add" -> {
                        data.put(idCounter++, name);
                        msg = "Added Successfully!";
                    }
                    case "delete" -> {
                        data.remove(id);
                        msg = "Deleted Successfully!";
                    }
                }
            }

            StringBuilder table = new StringBuilder("<table border='1' style='margin:auto'><tr><th>ID</th><th>Name</th></tr>");
            for (var e : data.entrySet()) {
                table.append("<tr><td>").append(e.getKey()).append("</td><td>").append(e.getValue()).append("</td></tr>");
            }
            table.append("</table>");

            String html = """
                <html>
                <head><title>Simple CRUD App</title></head>
                <body style="font-family:Arial;text-align:center;">
                    <h1>CRUD Application</h1>
                    <form method="GET" action="/">
                        <input type="hidden" name="action" value="add">
                        <label>Name:</label><input type="text" name="name">
                        <input type="submit" value="Add">
                    </form><br>
            """ + table + "<br>" + msg + "</body></html>";

            out.write("HTTP/1.1 200 OK\r\n");
            out.write("Content-Type: text/html\r\n");
            out.write("Content-Length: " + html.length() + "\r\n\r\n");
            out.write(html);
            out.flush();
            socket.close();
        }
    }
}


2)Dockerfile

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY build/CRUDServer.class /app/
EXPOSE 9092
CMD ["java", "CRUDServer"]




commands to run

javac src\CRUDServer.java -d build
docker build -t java-crud-server .
docker run -p 9092:9092 java-crud-server



4.Fetching information from API and print the same.



structure:

JavaAPIFetchApp/
├── src/
│   ├── APIServer.java
├── Dockerfile
└── README.md




1)BMIServer.java

import java.io.*;
import java.net.*;

public class APIServer {
    public static void main(String[] args) throws IOException {
        int port = 9093;
        ServerSocket serverSocket = new ServerSocket(port);
        System.out.println("API Fetch Server running on http://localhost:" + port);

        while (true) {
            Socket socket = serverSocket.accept();
            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            BufferedWriter out = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));

            String requestLine;
            while ((requestLine = in.readLine()) != null && !requestLine.isEmpty());

            // Fetch sample public API data
            URL url = new URL("https://api.github.com");
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            conn.setRequestMethod("GET");

            BufferedReader apiReader = new BufferedReader(new InputStreamReader(conn.getInputStream()));
            StringBuilder apiResponse = new StringBuilder();
            String line;
            while ((line = apiReader.readLine()) != null) apiResponse.append(line);
            apiReader.close();

            String html = """
                <html>
                <head><title>API Data Fetch</title></head>
                <body style="font-family:Arial;text-align:center;">
                    <h1>Fetched API Data from GitHub</h1>
                    <pre style="text-align:left;margin:auto;width:80%;border:1px solid #ccc;padding:10px;">""" + apiResponse + "</pre></body></html>";

            out.write("HTTP/1.1 200 OK\r\n");
            out.write("Content-Type: text/html\r\n");
            out.write("Content-Length: " + html.length() + "\r\n\r\n");
            out.write(html);
            out.flush();
            socket.close();
        }
    }
}


2)Dockerfile

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY build/APIServer.class /app/
EXPOSE 9093
CMD ["java", "APIServer"]



commands to run

javac src\APIServer.java -d build
docker build -t java-api-server .
docker run -p 9093:9093 java-api-server







II) JENKINS:


1)Jenkinsfile

pipeline {
    agent any

    environment {
        APP_NAME = 'JavaCalculatorApp'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Fetching source code from repository...'
                // If using GitHub or Git, uncomment below line:
                // git 'https://github.com/your-repo/java-calculator.git'
                echo 'Source code checkout complete.'
            }
        }

        stage('Build') {
            steps {
                echo 'Building the project... 🏗️'
                bat 'javac Calculator.java'
                echo 'Build completed successfully.'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests... 🧪'
                bat 'java Calculator'
                echo 'All tests executed successfully.'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying the application... 🚀'
                // You can add Docker or file copy commands here if needed
                echo 'Deployment successful! ✅'
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline completed successfully for ${APP_NAME}!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs for errors."
        }
    }
}


2)structure

JavaCalculatorApp/
├── Calculator.java
└── Jenkinsfile


3)calculator.java

public class Calculator {
    public static void main(String[] args) {
        int a = 10, b = 5;
        System.out.println("Basic Calculator Operations:");
        System.out.println("Addition: " + (a + b));
        System.out.println("Subtraction: " + (a - b));
        System.out.println("Multiplication: " + (a * b));
        System.out.println("Division: " + ((double)a / b));
        System.out.println("✅ Calculator executed successfully.");
    }
}


3)steps

Open Jenkins Dashboard → New Item → select Pipeline → name it e.g., JavaCalculatorPipeline.

Under Pipeline Definition, choose:

“Pipeline script from SCM” (if using GitHub)

or “Pipeline script” (and paste the Jenkinsfile code above).

Click Build Now.

You’ll see Jenkins executing:

Build → javac

Test → java

Deploy → simulated step



4)output

[Pipeline] echo
Building the project... 🏗️
[Pipeline] bat
E:\Jenkins\workspace>javac Calculator.java
[Pipeline] echo
Running tests... 🧪
[Pipeline] bat
E:\Jenkins\workspace>java Calculator
Basic Calculator Operations:
Addition: 15
Subtraction: 5
Multiplication: 50
Division: 2.0
✅ Calculator executed successfully.
[Pipeline] echo
Deployment successful! ✅
🎉 Pipeline completed successfully for JavaCalculatorApp!





III) GITHUB

1) In Folder -> shift+rigth click -> powershell

2)commands

git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/<your-username>/JavaCalculatorApp.git
git branch -M main
git push -u origin main




