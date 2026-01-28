# Bric RCTC Project

## Project Overview

This project showcases a complete enterprise application architecture with:
- **JakartaEE 10** platform
- **Apache Struts 7** MVC framework with Tiles layout management
- **Stateless EJB** session beans for business logic
- **Remote EJB** interface for distributed calls
- **JPA 3.1** with Hibernate and DB2
- **JDBC alternative** implementation with repository pattern
- **Security integration** between web and EJB layers
- **Maven multi-module** project structure
- **EAR packaging** for enterprise deployment

## Migration status

The following modules are fully migrated:

- VioEJB
- VioUtil
- VioWeb
- VioWebAdmin
- VTX-TXAPI
- VioTaskClient
- VioLetterClient
- VioBatchClient
- VioApp

## Prerequisites

- **Java 17** or higher
- **Maven 3.8+** (or use the included Maven wrapper)
- **DB2 Database**
- **TomEE 10.0+ Plus** or **GlassFish 7**

### Maven Wrapper

This project includes a Maven wrapper, so you don't need to have Maven installed. You can use:

- **Linux/macOS**: `./mvnw` instead of `mvn`
- **Windows**: `mvnw.cmd` instead of `mvn`

All Maven commands in this README can be executed with either `mvn` or `./mvnw`.

## Build Instructions

### Build the entire project:

Using installed Maven:

```bash
mvn clean install
```

Or using Maven wrapper (no Maven installation required):

```bash
# Linux/macOS
./mvnw clean install

# Windows
mvnw.cmd clean install
```

This will:
1. Compile all modules
2. Run tests (if any)
3. Package modules (JAR, EJB, WAR)
4. Create the EAR file at `VioApp/target/VioApp.ear`

## Deployment Instructions

### Option 1: Apache TomEE 10.0+ Plus

#### 1. Download and Install TomEE

```bash
# Download TomEE 10.0+ Plus
wget https://dlcdn.apache.org/tomee/tomee-10.0.0-M3/apache-tomee-10.0.0-M3-plus.tar.gz
tar -xzf apache-tomee-10.0.0-M3-plus.tar.gz
cd apache-tomee-plus-10.0.0-M3
```

#### 2. Configure DataSource

Edit `conf/tomee.xml` and add:

```xml
<tomee>
  <Resource id="jdbc/test_rctc_db" type="DataSource">
    JdbcDriver com.ibm.db2.jcc.DB2Driver
    JdbcUrl jdbc:db2://SERVER:50001/rctc_db:currentSchema=rctcinst;
    UserName DBUSER
    Password DBPWD
    JtaManaged true
    InitialSize 5
    MaxActive 20
    MaxIdle 10
    MinIdle 5
    MaxWait 10000
    TestOnBorrow true
    ValidationQuery SELECT 1 FROM SYSIBM.SYSDUMMY1
  </Resource>
</tomee>
```

#### 3. Copy DB2 JDBC Driver

```bash
# Download DB2 JDBC driver
wget https://repo1.maven.org/maven2/com/ibm/db2/jcc/11.1.4.4/jcc-11.1.4.4.jar -P lib/

# or Download PostgreSQL JDBC driver
wget https://repo1.maven.org/maven2/org/postgresql/postgresql/42.7.9/postgresql-42.7.9.jar -P lib/
```

#### 4. Deploy the EAR

```bash
# Copy EAR to apps directory
cp VioApp/target/VioApp.ear apps/
```

#### 5. Configure Security Realm (Optional)

Edit `conf/tomcat-users.xml` to add users:

```xml
  <role rolename="admin" />
  <role rolename="user" />
  <user username="admin" password="adminpass" roles="admin,user" />
  <user username="user" password="userpass" roles="user" />
```

#### 6. Start TomEE

```bash
bin/startup.sh  # Linux/macOS
bin/startup.bat # Windows

# View logs
tail -f logs/catalina.out
```

#### 7. Access the Application

Open browser: `http://localhost:8080/Violation/`

---

### Option 2: GlassFish 7

#### 1. Download and Install GlassFish

```bash
# Download GlassFish 7
wget https://download.eclipse.org/ee4j/glassfish/glassfish-7.0.14.zip
unzip glassfish-7.0.14.zip
cd glassfish7
```

#### 2. Start GlassFish

```bash
bin/asadmin start-domain domain1
```

#### 3. Configure PostgreSQL JDBC Driver

```bash
# Copy DB2 driver to GlassFish lib
wget https://repo1.maven.org/maven2/com/ibm/db2/jcc/11.1.4.4/jcc-11.1.4.4.jar -P glassfish/domains/domain1/lib/
```

#### 4. Create JDBC Connection Pool

```bash
bin/asadmin create-jdbc-connection-pool \
  --datasourceclassname org.postgresql.ds.PGConnectionPoolDataSource \
  --restype javax.sql.ConnectionPoolDataSource \
  --property "user=sampleuser:password=samplepass:serverName=localhost:portNumber=5432:databaseName=sampledb" \
  SamplePool
```

#### 5. Create JDBC Resource

```bash
bin/asadmin create-jdbc-resource \
  --connectionpoolid SamplePool \
  jdbc/SampleDS
```

#### 6. Configure Security Realm (Optional)

```bash
# Create file realm
bin/asadmin create-auth-realm \
  --classname com.sun.enterprise.security.auth.realm.file.FileRealm \
  --property file=${com.sun.aas.instanceRoot}/config/keyfile:jaas-context=fileRealm \
  sample-realm

# Add users
bin/asadmin create-file-user \
  --groups admin,user \
  --authrealmname sample-realm \
  admin

bin/asadmin create-file-user \
  --groups user \
  --authrealmname sample-realm \
  user
```

#### 7. Deploy the EAR

```bash
# Via command line
bin/asadmin deploy VioApp/target/VioApp.ear

# Or via Admin Console
# Open http://localhost:4848
# Applications -> Deploy -> Upload VioApp.ear
```

#### 8. Access the Application

Open browser: `http://localhost:8080/Violation`

---

### Security Configuration

The application implements declarative security:

1. **EJB Layer**:
   - Role-based access control with `@RolesAllowed`, `@PermitAll`
   - Security context propagation from web to EJB layer
   - Configured in `ejb-jar.xml`

2. **Web Layer**:
   - Form-based authentication
   - Protected resources under `/secure/*`
   - Configured in `web.xml`

### Struts 7 Plugins

The following Struts 7 plugins are configured:

1. **Convention Plugin** - Zero-configuration using naming conventions
2. **Tiles Plugin** - Layout management and templating
3. **JSON Plugin** - REST API support (optional)
4. **CDI Plugin** - Integration with JakartaEE CDI for dependency injection

#### Port Conflicts

**Solution**:
```bash
# Change TomEE ports in conf/server.xml
<Connector port="8080" ... />  # Change to 8081, etc.

# Change GlassFish ports
bin/asadmin set configs.config.server-config.http-service.http-listener.http-listener-1.port=8081
```

### Debugging

Enable detailed logging:

**TomEE**: Edit `conf/logging.properties`
```properties
.level = INFO
com.example.jakartaee.level = FINE
org.apache.struts2.level = FINE
```

**GlassFish**: Edit `glassfish/domains/domain1/config/logging.properties`
```properties
com.example.jakartaee.level=FINE
org.apache.struts2.level=FINE
```

## Additional Resources

- [JakartaEE Documentation](https://jakarta.ee/)
- [Apache Struts Documentation](https://struts.apache.org/)
- [TomEE Documentation](https://tomee.apache.org/)
- [GlassFish Documentation](https://glassfish.org/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Awesome Jakarta EE Resources for Developers](https://awesome-jakartaee.github.io/)


------

# Using AI in development

## SpecKit

Spec-Driven Development flips the script on traditional software development. For decades, code has been king — specifications were just scaffolding we built and discarded once the "real work" of coding began. Spec-Driven Development changes this: specifications become executable, directly generating working implementations rather than just guiding them.

Note: the GitHub SpecKit works with any Cli AI tools (Claude Code, Gemini CLI, OpenAI Codex etc.)

### Installation

```bash
Install uv tool:
curl -LsSf https://astral.sh/uv/install.sh | sh

Install SpecKit:
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git

or upgrade:
uv tool install specify-cli --force --from git+https://github.com/github/spec-kit.git
```

then:

```bash
specify init .
specify check
```

Documentation:
https://github.com/github/spec-kit

Specify commands available:
1. /speckit.constitution - Establish project principles
2. /speckit.specify - Create baseline specification
3. /speckit.clarify (optional) - Ask structured questions to de-risk ambiguous areas before planning (run before /speckit.plan if used)
4. /speckit.plan - Create implementation plan
5. /speckit.checklist (optional) - Generate quality checklists to validate requirements completeness, clarity, and consistency (after /speckit.plan)
6. /speckit.tasks - Generate actionable tasks
7. /speckit.analyze (optional) - Cross-artifact consistency & alignment report (after /speckit.tasks, before /speckit.implement)
8. /speckit.implement - Execute implementation

## Claude Code

### Installation

```bash
$ npm install -g @anthropic-ai/claude-code
$ npm install -g @anthropic-ai/claude-code@latest
$ claude
or
$ claude --dangerously-skip-permissions
```

OR

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

In Visual Studion Code -> Extensions (Ctrl+Shift+x) -> Search and add extension: Anthropic.claude-code

Claud Desktop: https://docs.pieces.app/products/mcp/claude-desktop

## Gemini CLI

```bash
npm install -g @google/gemini-cli
```

## Claude Desktop

Windows/MacOS: https://claude.com/download

Linux;
https://github.com/aaddrick/claude-desktop-debian/releases
https://github.com/aaddrick/claude-desktop-debian/releases/download/v1.1.10%2Bclaude0.14.10/claude-desktop-0.14.10-amd64.AppImage

## Improving your development experience with TomEE 10 in VS Code:

Start TomEE with Hot Reload

1. Build with debug information:
./mvnw clean package -DskipTests,tomee

2. Run/Debug
./mvnw org.apache.tomee.maven:tomee-maven-plugin:debug -pl VioApp -Dtomee

Utilities
- Check active profiles: ./mvnw help:active-profiles -Dtomee -pl VioApp

## Java classes Hot Swap Configuration

HotswapAgent + DCEVM Setup Guide for TomEE (Java 21)

 What is DCEVM + HotswapAgent?

 DCEVM (Dynamic Code Evolution VM) is a patched JVM that extends hot-swap capabilities beyond standard Java limitations:

 | Feature                | Standard JVM | DCEVM |
 |------------------------|--------------|-------|
 | Change method body     | ✅           | ✅    |
 | Add new method         | ❌           | ✅    |
 | Remove method          | ❌           | ✅    |
 | Add/remove fields      | ❌           | ✅    |
 | Change class hierarchy | ❌           | ❌    |

 HotswapAgent is a plugin framework that bridges DCEVM with application frameworks. It handles:
 - Reinitializing CDI beans after class change
 - Refreshing Struts action mappings
 - Reloading EJB proxies
 - And more...

 Why This Solves Your Problem

 Your current issue:
 .class file changed on disk → TomEE classloader ignores it → old code runs

 With DCEVM + HotswapAgent:
 .class file changed → DCEVM redefines class in JVM → HotswapAgent refreshes CDI/Struts → new code runs instantly

 No context reload. No session loss. Sub-second turnaround.

 ---
 Setup Steps

 Step 1: Download JetBrains Runtime (JBR) with DCEVM

 JetBrains Runtime is a patched OpenJDK that includes DCEVM support.

 1. Go to https://github.com/JetBrains/JetBrainsRuntime/releases
 2. Download JBR 21 for your platform (look for jbr_dcevm-21.x.x-linux-x64 or similar)
 3. Extract to a location, e.g., /opt/jbr-21

 # Example for Linux x64
 cd /opt
 wget https://github.com/JetBrains/JetBrainsRuntime/releases/download/jbr-release-21.0.5b631.8/jbr_dcevm-21.0.5-linux-x64-b631.8.tar.gz
 tar -xzf jbr_dcevm-21.0.5-linux-x64-b631.8.tar.gz
 mv jbr_dcevm-21.0.5-linux-x64-b631.8 jbr-21

 Step 2: Download HotswapAgent

 1. Go to https://github.com/HotswapProjects/HotswapAgent/releases
 2. Download hotswap-agent-X.X.X.jar (latest version)
 3. Important: Rename to exactly hotswap-agent.jar
 4. Copy to JBR's lib/hotswap folder:

 mkdir -p /opt/jbr-21/lib/hotswap
 cp hotswap-agent-*.jar /opt/jbr-21/lib/hotswap/hotswap-agent.jar

 Step 3: Create hotswap-agent.properties

 Create a configuration file in your project's classpath root (e.g., VioWeb/src/hotswap-agent.properties):

 # Enable plugins for your stack
 pluginPackages=org.hotswap.agent.plugin

 # Disable plugins you don't need (speeds up startup)
 disabledPlugins=Spring,Hibernate,Vaadin

 # Enable useful plugins for your stack
 # CDI (OpenWebBeans - used by TomEE)
 owb.plugin=true

 # EJB support
 ejb.plugin=true

 # Watch extra directories for changes
 extraClasspath=/workspaces/VioWeb/target/classes;\
 /workspaces/VioEJB/target/classes;\
 /workspaces/VioUtil/target/classes

 # Logging
 LOGGER=info

 Step 4: Configure TomEE Maven Plugin to Use JBR

 Modify VioApp/pom.xml in the tomee profile to use JBR and enable DCEVM:

 <plugin>
     <groupId>org.apache.tomee.maven</groupId>
     <artifactId>tomee-maven-plugin</artifactId>
     <configuration>
         <!-- Use JBR with DCEVM -->
         <args>-XX:+AllowEnhancedClassRedefinition -XX:HotswapAgent=fatjar</args>

         <!-- Keep reloadOnUpdate false - DCEVM handles it -->
         <reloadOnUpdate>false</reloadOnUpdate>

         <!-- ... rest of your config ... -->
     </configuration>
 </plugin>

 Step 5: Set JAVA_HOME to JBR

 Before running Maven:

 export JAVA_HOME=/opt/jbr-21
 export PATH=$JAVA_HOME/bin:$PATH

 # Verify
 java -version
 # Should show JetBrains Runtime

 Or add to your .bashrc / .zshrc / devcontainer.json.

 Step 6: Run and Test

 # Build
 ./mvnw clean compile -pl VioWeb -am -Dtomee

 # Start TomEE with HotswapAgent
 JAVA_HOME=/opt/jbr-21 ./mvnw org.apache.tomee.maven:tomee-maven-plugin:debug -pl VioApp -Dtomee

 Now when you change SearchBatchForm.java:
 1. VS Code auto-compiles to .class
 2. tomee-maven-plugin syncs to deployed location (or debugger pushes via JPDA)
 3. DCEVM redefines the class in the running JVM
 4. HotswapAgent refreshes CDI beans
 5. Refresh browser → new code runs immediately

 ---
 Relevant Plugins for Your Stack

 | Plugin       | Purpose                   | Your Use                 |
 |--------------|---------------------------|--------------------------|
 | OpenWebBeans | CDI bean reloading        | TomEE uses OWB for CDI   |
 | EJB          | Session bean reloading    | Your VioEJB module       |
 | Tomcat       | Web container integration | TomEE is based on Tomcat |
 | Logback      | Log config reload         | If using Logback         |

 Plugin Configuration Example

 # VioWeb/src/hotswap-agent.properties

 # OpenWebBeans (CDI) - critical for your Struts actions with @Inject
 owb.plugin=true

 # EJB support for VioEJB
 ejb.plugin=true

 # Watch these directories for .class changes
 extraClasspath=/workspaces/VioWeb/target/classes;\
 /workspaces/VioEJB/target/classes

 # Tomcat integration (handles web.xml, etc.)
 tomcat.plugin=true

 ---
 Limitations

 1. Cannot change class hierarchy (extends/implements)
 2. Cannot add/remove enum constants
 3. Cannot change annotations on injected fields (requires container restart)
 4. Some framework caches may need manual refresh

 ---
 Alternative: VS Code Debug Hot-Swap (Simpler but Limited)

 If DCEVM setup is too complex, VS Code's debugger can do basic hot-swap:

 1. Connect debugger to TomEE (already working for you)
 2. Edit Java code
 3. Save file → VS Code compiles
 4. Use "Hot Code Replace" (automatic in debug mode)

 Limitation: Only method body changes. No new methods/fields.

 This explains why your debugger shows new value but UI doesn't - the JVM replaced bytecode, but Struts/CDI still holds references to the old object state.

 ---
 Summary

 | Component                           | Purpose                                                            |
 |-------------------------------------|--------------------------------------------------------------------|
 | JBR (JetBrains Runtime)             | JVM with DCEVM patches for enhanced class redefinition             |
 | HotswapAgent                        | Plugin framework that refreshes framework state after class reload |
 | hotswap-agent.properties            | Configuration for which plugins to enable                          |
 | -XX:+AllowEnhancedClassRedefinition | JVM flag to enable DCEVM                                           |
 | -XX:HotswapAgent=fatjar             | JVM flag to enable HotswapAgent with all plugins                   |

 Sources

 - https://github.com/HotswapProjects/HotswapAgent
 - https://github.com/JetBrains/JetBrainsRuntime
 - http://dcevm.github.io/
