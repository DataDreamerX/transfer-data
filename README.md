import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.util.List;

@Service
public class McpServerService {

    private final McpServerRepository mcpServerRepository;

    @Autowired
    public McpServerService(McpServerRepository mcpServerRepository) {
        this.mcpServerRepository = mcpServerRepository;
    }

    /**
     * Create a new server (local or remote).
     * The polymorphic mapping is handled by Jackson before this method is even called.
     */
    @Transactional
    public McpServer createServer(McpServer server) {
        // You could add validation logic here, e.g., check for duplicate names
        return mcpServerRepository.save(server);
    }

    /**
     * Get a list of all servers.
     */
    public List<McpServer> getAllServers() {
        return mcpServerRepository.findAll();
    }

    /**
     * Get a single server by its ID.
     */
    public McpServer getServerById(Long id) {
        return mcpServerRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("McpServer not found with id: " + id));
    }

    /**
     * Delete a server by its ID.
     */
    @Transactional
    public void deleteServer(Long id) {
        McpServer server = getServerById(id); // Ensures server exists before deleting
        mcpServerRepository.delete(server);
    }

    /**
     * Update an existing server.
     * This logic is more complex because it must handle the two subtypes.
     */
    @Transactional
    public McpServer updateServer(Long id, McpServer serverDetails) {
        McpServer existingServer = getServerById(id);

        // Basic check: Don't allow changing the server type (e.g., local -> remote) via update.
        if (!existingServer.getClass().equals(serverDetails.getClass())) {
            throw new IllegalArgumentException("Cannot change server type. Please delete and recreate.");
        }

        // --- Update common fields ---
        existingServer.setServerName(serverDetails.getServerName());
        existingServer.setAutoStart(serverDetails.isAutoStart());

        // --- Update type-specific fields ---
        if (existingServer instanceof LocalMcpServer existingLocalServer) {
            LocalMcpServer localDetails = (LocalMcpServer) serverDetails;
            existingLocalServer.setCommand(localDetails.getCommand());
            existingLocalServer.setArgs(localDetails.getArgs());
            existingLocalServer.setEnv(localDetails.getEnv());
        } 
        else if (existingServer instanceof RemoteMcpServer existingRemoteServer) {
            RemoteMcpServer remoteDetails = (RemoteMcpServer) serverDetails;
            existingRemoteServer.setServerUrl(remoteDetails.getServerUrl());
            existingRemoteServer.setBearerToken(remoteDetails.getBearerToken());
            existingRemoteServer.setTransportType(remoteDetails.getTransportType());
        }

        return mcpServerRepository.save(existingServer);
    }
}




import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import jakarta.validation.Valid;
import java.util.List;

@RestController
@RequestMapping("/api/mcp-servers") // Base path for all endpoints in this controller
public class McpServerController {

    private final McpServerService mcpServerService;

    @Autowired
    public McpServerController(McpServerService mcpServerService) {
        this.mcpServerService = mcpServerService;
    }

    /**
     * POST /api/mcp-servers
     * Create a new MCP Server.
     * Jackson will read the "type" field and create the correct object.
     */
    @PostMapping
    public ResponseEntity<McpServer> createMcpServer(@Valid @RequestBody McpServer server) {
        McpServer createdServer = mcpServerService.createServer(server);
        return new ResponseEntity<>(createdServer, HttpStatus.CREATED);
    }

    /**
     * GET /api/mcp-servers
     * Get all MCP Servers.
     */
    @GetMapping
    public List<McpServer> getAllMcpServers() {
        return mcpServerService.getAllServers();
    }

    /**
     * GET /api/mcp-servers/{id}
     * Get a single MCP Server by its ID.
     */
    @GetMapping("/{id}")
    public ResponseEntity<McpServer> getMcpServerById(@PathVariable(value = "id") Long id) {
        McpServer server = mcpServerService.getServerById(id);
        return ResponseEntity.ok(server);
    }

    /**
     * PUT /api/mcp-servers/{id}
     * Update an existing MCP Server.
     */
    @PutMapping("/{id}")
    public ResponseEntity<McpServer> updateMcpServer(@PathVariable(value = "id") Long id,
                                                    @Valid @RequestBody McpServer serverDetails) {
        McpServer updatedServer = mcpServerService.updateServer(id, serverDetails);
        return ResponseEntity.ok(updatedServer);
    }

    /**
     * DELETE /api/mcp-servers/{id}
     * Delete an MCP Server.
     */
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteMcpServer(@PathVariable(value = "id") Long id) {
        mcpServerService.deleteServer(id);
        return ResponseEntity.noContent().build(); // HTTP 204
    }
}
