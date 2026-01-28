# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RCTC (Rent To Clear) Violation management system - an enterprise JakartaEE 10 application running on TomEE 10.1.2. This project is a migration from J2EE+Struts 1.3 to JakartaEE 10+Struts 2 (version 7).

**Fully migrated modules:** VioEJB, VioUtil, VioWeb, VioWebAdmin, VTX-TXAPI, VioTaskClient, VioLetterClient, VioBatchClient, VioApp

## Build Commands

```bash
# Full build (compile, test, package)
./mvnw clean install

# Compile only
./mvnw clean compile

# Run unit tests
./mvnw test

# Run single test class
./mvnw test -Dtest=ClassName

# Run single test method
./mvnw test -Dtest=ClassName#methodName

# Run tests for specific module
./mvnw -pl VioUtil test

# Package without tests
./mvnw package -DskipTests

# Run integration tests (disabled by default)
./mvnw verify -DskipITs=false
```

## Architecture

### Multi-Tier EAR Structure

```
VioApp.ear (rctc.ear)
├── VioUtil.jar          - Shared utilities, DTOs, resources
├── VioEJB.jar           - EJB 4.0 session beans, domain objects
├── VioWeb.war           - Main web app (/Violation context)
├── VioWebAdmin.war      - Admin web app (/VioAdmin context)
└── VTX-TXAPI.war        - REST API (/tx context)
```

### Module Purposes

| Module | Type | Description |
|--------|------|-------------|
| VioUtil | jar | Shared utilities, DTOs, resource bundles |
| VioEJB | ejb | Business logic in EJB 4.0 session beans |
| VioWeb | war | Main Struts 2/7 MVC web application |
| VioWebAdmin | war | Administrative interface |
| VTX-TXAPI | war | REST/JSON API endpoints |
| VioTaskClient | jar | Remote EJB client for task management |
| VioBatchClient | jar | Remote EJB client for batch jobs |
| VioLetterClient | jar | Remote EJB client for letter generation |

### Key Source Locations

- **Struts Actions:** `VioWeb/src/com/vesystems/violation/struts/`
- **EJB Session Beans:** `VioEJB/ejbModule/com/vesystems/violation/ejb/`
- **Domain Objects:** `VioEJB/ejbModule/com/vesystems/violation/domainIP/`
- **DTOs:** `VioUtil/src/com/vesystems/violation/dto/`
- **Struts Config:** `VioWeb/src/struts.xml`
- **Tiles Layout:** `VioWeb/WebContent/WEB-INF/tiles.xml`
- **JSP Pages:** `VioWeb/WebContent/jsp/`

### Technology Stack

- **Java:** 21
- **JakartaEE:** 10.0.0 (jakarta.* namespace)
- **Struts:** 2 v7.0.3 with Convention, Tiles, JSON, CDI plugins
- **App Server:** TomEE Plus 10.1.2
- **EJB:** 4.0 (session beans)
- **Database:** IBM DB2
- **Build:** Maven 3.8.7 (via wrapper)
- **Testing:** JUnit 5

### Dependency Injection Pattern

Struts 2 uses CDI for dependency injection. EJBs are injected into Actions:

```java
public class SomeAction extends ActionSupport {
    @Inject
    private SomeSessionBean sessionBean;
}
```

CDI is enabled via `/WEB-INF/beans.xml` and Struts uses `struts.objectFactory=cdi`.

## Development Environment

Uses VS Code Dev Containers with Docker Compose.

### Custom Source Directories

This project uses non-standard Maven source directories (legacy Eclipse structure):

| Module | Source Directory | Resources |
|--------|-----------------|-----------|
| VioUtil | `src` | `src` |
| VioEJB | `ejbModule` | `ejbModule` |
| VioWeb | `src` | `WebContent` |

### Running and Debugging with TomEE Maven Plugin

The project uses the TomEE Maven Plugin for development with hot-reload support. The plugin is configured in `VioWeb/pom.xml`.

**Start TomEE (no debug):**
```bash
./mvnw compile -pl VioWeb -am && ./mvnw org.apache.tomee.maven:tomee-maven-plugin:run -pl VioWeb
```

**Start TomEE with debug (port 5005):**
```bash
./mvnw compile -pl VioWeb -am && ./mvnw org.apache.tomee.maven:tomee-maven-plugin:debug -pl VioWeb
```

**Key flags:**
- `-pl VioWeb` - Run the TomEE plugin only on VioWeb module
- `-am` (also-make) - Automatically compile dependent modules (VioUtil, VioEJB)

Note: The full plugin goal name (`org.apache.tomee.maven:tomee-maven-plugin:run`) is required because the `tomee:` prefix is not in Maven's default plugin groups.

**Access the application:** http://localhost:8080/Violation/

**Hot Reload Behavior:**

| File Type | Behavior |
|-----------|----------|
| `.jsp`, `.html`, `.css`, `.js`, images | Synced immediately, no restart |
| `.properties`, `.xml` | Synced immediately, no restart |
| `.class` files | Triggers context reload |

The plugin checks for changes every 2 seconds.

**Development Workflow:**
1. Start TomEE: `./mvnw compile tomee:debug -pl VioWeb -am`
2. Attach VS Code debugger (F5 → "TomEE Debug (Hot Reload)")
3. Edit JSP/CSS/JS → Changes appear on browser refresh
4. Edit Java code → Run `./mvnw compile -pl VioWeb -am` in another terminal
5. The plugin detects `.class` changes and reloads the context automatically

### VS Code Launch Configurations

- **TomEE Debug (Hot Reload)** - Compiles, starts TomEE in debug mode, and attaches debugger
- **TomEE Attach Only** - Attaches to an already running TomEE debug session

### Web App Context Roots

- `/Violation` - Main application (VioWeb)
- `/VioAdmin` - Admin interface (VioWebAdmin)
- `/tx` - REST API (VTX-TXAPI)

### Port Conflict Note

If the external TomEE at `/opt/tomcat` is running on port 8080, stop it first:
```bash
/opt/tomcat/bin/shutdown.sh
```

## Important Notes

- **Parent POM:** `VioApp/MavenParent/pom.xml` manages all dependency versions
- **Local libs:** Obsolete/custom libraries in `VioApp/libs/` as file-based Maven repo
- **Integration tests:** Disabled by default (`skipITs=true`)
- **Non-skinny WARs:** Each WAR includes its own dependencies (not extracted to EAR lib)
- **Legacy code:** Modules with `-old` suffix are deprecated migration artifacts
