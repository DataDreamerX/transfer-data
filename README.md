import com.fasterxml.jackson.annotation.JsonSubTypes;
import com.fasterxml.jackson.annotation.JsonTypeInfo;
import jakarta.validation.constraints.NotBlank;

/**
 * Base abstract class for any MCP Server configuration.
 * Uses Jackson's polymorphism to handle different server types.
 */
@JsonTypeInfo(
        use = JsonTypeInfo.Id.NAME,
        include = JsonTypeInfo.As.PROPERTY,
        property = "type" // This JSON field will determine the subclass
)
@JsonSubTypes({
        @JsonSubTypes.Type(value = LocalMcpServer.class, name = "local"),
        @JsonSubTypes.Type(value = RemoteMcpServer.class, name = "remote")
})
public abstract class McpServer {

    @NotBlank(message = "Server Name is required.")
    private String serverName;

    private boolean autoStart = false;

    // Getters and Setters
    public String getServerName() {
        return serverName;
    }

    public void setServerName(String serverName) {
        this.serverName = serverName;
    }

    public boolean isAutoStart() {
        return autoStart;
    }

    public void setAutoStart(boolean autoStart) {
        this.autoStart = autoStart;
    }
}


import com.fasterxml.jackson.annotation.JsonTypeName;
import jakarta.validation.constraints.NotBlank;
import java.util.List;
import java.util.Map;

/**
 * Represents a locally managed MCP server that is started with a command.
 * Corresponds to the "Create Manually" tab.
 */
@JsonTypeName("local") // Maps to type = "local" in the JSON
public class LocalMcpServer extends McpServer {

    @NotBlank(message = "Command is required for local servers.")
    private String command;

    // Use a List for arguments, as shown in your JSON example
    private List<String> args;

    // Use a Map for environment variables
    private Map<String, String> env;

    // Getters and Setters
    public String getCommand() {
        return command;
    }

    public void setCommand(String command) {
        this.command = command;
    }

    public List<String> getArgs() {
        return args;
    }

    public void setArgs(List<String> args) {
        this.args = args;
    }

    public Map<String, String> getEnv() {
        return env;
    }

    public void setEnv(Map<String, String> env) {
        this.env = env;
    }
}



import com.fasterxml.jackson.annotation.JsonTypeName;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import org.hibernate.validator.constraints.URL;

/**
 * Represents a connection to a remote MCP server.
 * Corresponds to the "Remote Server" tab.
 */
@JsonTypeName("remote") // Maps to type = "remote" in the JSON
public class RemoteMcpServer extends McpServer {

    @NotBlank(message = "Server URL is required.")
    @URL(message = "Must be a valid URL.")
    private String serverUrl;

    // This could be sensitive, handle with care
    private String bearerToken;

    @NotNull(message = "Transport Type is required.")
    private TransportType transportType;

    // Getters and Setters
    public String getServerUrl() {
        return serverUrl;
    }

    public void setServerUrl(String serverUrl) {
        this.serverUrl = serverUrl;
    }

    public String getBearerToken() {
        return bearerToken;
    }

    public void setBearerToken(String bearerToken) {
        this.bearerToken = bearerToken;
    }

    public TransportType getTransportType() {
        return transportType;
    }

    public void setTransportType(TransportType transportType) {
        this.transportType = transportType;
    }
}



import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;
import jakarta.validation.Valid;

@RestController
public class McpServerController {

    @PostMapping("/api/mcp-servers")
    public ResponseEntity<?> addMcpServer(@Valid @RequestBody McpServer server) {
        
        // Spring will have already created either a LocalMcpServer or RemoteMcpServer
        
        if (server instanceof LocalMcpServer localServer) {
            // Logic to save or start the local server
            System.out.println("Adding new local server: " + localServer.getServerName());
            System.out.println("Command: " + localServer.getCommand());

        } else if (server instanceof RemoteMcpServer remoteServer) {
            // Logic to connect to and save the remote server
            System.out.println("Adding new remote server: " + remoteServer.getServerName());
            System.out.println("URL: " + remoteServer.getServerUrl());
            System.out.println("Transport: " + remoteServer.getTransportType());
        }

        // ... save the server to your database or service ...
        
        return ResponseEntity.ok("Server " + server.getServerName() + " added.");
    }
}
