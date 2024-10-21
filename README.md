# java-postgre-sql-db

Sure, let's break this down step-by-step. We'll start by setting up Docker Compose with PostgreSQL and pgAdmin, then connect it with a Java application. Finally, we'll create a user table and implement CRUD operations in Java.

### Step 1: Set Up Docker Compose with PostgreSQL and pgAdmin

1. **Create a `docker-compose.yml` file:**

    ```yaml
    version: '3.8'
    services:
      postgres:
        image: postgres:latest
        container_name: postgres
        environment:
          POSTGRES_USER: admin
          POSTGRES_PASSWORD: admin
          POSTGRES_DB: testdb
        ports:
          - "5432:5432"
        volumes:
          - postgres_data:/var/lib/postgresql/data

      pgadmin:
        image: dpage/pgadmin4
        container_name: pgadmin
        environment:
          PGADMIN_DEFAULT_EMAIL: admin@admin.com
          PGADMIN_DEFAULT_PASSWORD: admin
        ports:
          - "8080:80"
        depends_on:
          - postgres

    volumes:
      postgres_data:
    ```

2. **Start the Docker containers:**

    ```sh
    docker-compose up -d
    ```

### Step 2: Connect Java Application to PostgreSQL

1. **Add PostgreSQL dependency to your `pom.xml` (for Maven):**

    ```xml
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <version>42.2.23</version>
    </dependency>
    ```

2. **Create a Java class to manage the database connection:**

    ```java
    import java.sql.Connection;
    import java.sql.DriverManager;
    import java.sql.SQLException;

    public class DatabaseConnection {
        private static final String URL = "jdbc:postgresql://localhost:5432/testdb";
        private static final String USER = "admin";
        private static final String PASSWORD = "admin";

        public static Connection getConnection() throws SQLException {
            return DriverManager.getConnection(URL, USER, PASSWORD);
        }

        public static void main(String[] args) {
            try (Connection connection = getConnection()) {
                if (connection != null) {
                    System.out.println("Connected to the database!");
                } else {
                    System.out.println("Failed to make connection!");
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
    ```

### Step 3: Create User Table and Implement CRUD Operations

1. **Create the `User` table:**

    ```sql
    CREATE TABLE users (
        id SERIAL PRIMARY KEY,
        name VARCHAR(100),
        email VARCHAR(100) UNIQUE
    );
    ```

2. **Java class for CRUD operations:**

    ```java
    import java.sql.*;
    import java.util.ArrayList;
    import java.util.List;

    public class UserDAO {
        private Connection connection;

        public UserDAO() throws SQLException {
            this.connection = DatabaseConnection.getConnection();
        }

        public List<User> getAllUsers() throws SQLException {
            List<User> users = new ArrayList<>();
            String query = "SELECT * FROM users";
            try (Statement stmt = connection.createStatement();
                 ResultSet rs = stmt.executeQuery(query)) {
                while (rs.next()) {
                    users.add(new User(rs.getInt("id"), rs.getString("name"), rs.getString("email")));
                }
            }
            return users;
        }

        public User getUser(int id) throws SQLException {
            String query = "SELECT * FROM users WHERE id = ?";
            try (PreparedStatement pstmt = connection.prepareStatement(query)) {
                pstmt.setInt(1, id);
                try (ResultSet rs = pstmt.executeQuery()) {
                    if (rs.next()) {
                        return new User(rs.getInt("id"), rs.getString("name"), rs.getString("email"));
                    }
                }
            }
            return null;
        }

        public void addUser(User user) throws SQLException {
            String query = "INSERT INTO users (name, email) VALUES (?, ?)";
            try (PreparedStatement pstmt = connection.prepareStatement(query)) {
                pstmt.setString(1, user.getName());
                pstmt.setString(2, user.getEmail());
                pstmt.executeUpdate();
            }
        }

        public void addAllUsers(List<User> users) throws SQLException {
            String query = "INSERT INTO users (name, email) VALUES (?, ?)";
            try (PreparedStatement pstmt = connection.prepareStatement(query)) {
                for (User user : users) {
                    pstmt.setString(1, user.getName());
                    pstmt.setString(2, user.getEmail());
                    pstmt.addBatch();
                }
                pstmt.executeBatch();
            }
        }

        public void updateUser(User user) throws SQLException {
            String query = "UPDATE users SET name = ?, email = ? WHERE id = ?";
            try (PreparedStatement pstmt = connection.prepareStatement(query)) {
                pstmt.setString(1, user.getName());
                pstmt.setString(2, user.getEmail());
                pstmt.setInt(3, user.getId());
                pstmt.executeUpdate();
            }
        }

        public void updateAllUsers(List<User> users) throws SQLException {
            String query = "UPDATE users SET name = ?, email = ? WHERE id = ?";
            try (PreparedStatement pstmt = connection.prepareStatement(query)) {
                for (User user : users) {
                    pstmt.setString(1, user.getName());
                    pstmt.setString(2, user.getEmail());
                    pstmt.setInt(3, user.getId());
                    pstmt.addBatch();
                }
                pstmt.executeBatch();
            }
        }

        public void deleteUser(int id) throws SQLException {
            String query = "DELETE FROM users WHERE id = ?";
            try (PreparedStatement pstmt = connection.prepareStatement(query)) {
                pstmt.setInt(1, id);
                pstmt.executeUpdate();
            }
        }

        public void deleteAllUsers() throws SQLException {
            String query = "DELETE FROM users";
            try (Statement stmt = connection.createStatement()) {
                stmt.executeUpdate(query);
            }
        }
    }
    ```

3. **User class:**

    ```java
    public class User {
        private int id;
        private String name;
        private String email;

        public User(int id, String name, String email) {
            this.id = id;
            this.name = name;
            this.email = email;
        }

        // Getters and setters
    }
    ```

### Step 4: Insert Test Data

1. **Insert data using SQL:**

    ```sql
    INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com');
    INSERT INTO users (name, email) VALUES ('Jane Doe', 'jane@example.com');
    ```

2. **Insert data using Java:**

    ```java
    public static void main(String[] args) {
        try {
            UserDAO userDAO = new UserDAO();
            userDAO.addUser(new User(0, "John Doe", "john@example.com"));
            userDAO.addUser(new User(0, "Jane Doe", "jane@example.com"));

            List<User> users = userDAO.getAllUsers();
            users.forEach(user -> System.out.println(user.getName() + " - " + user.getEmail()));
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    ```

This should get you started with Docker Compose, PostgreSQL, pgAdmin, and a Java application with CRUD operations. Let me know if you need any more details or help with specific parts!

