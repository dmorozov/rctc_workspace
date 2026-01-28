# Bric RCTC Project

## Project Overview

This project is using JakartaEE enterprise application architecture with:
- **JakartaEE 10** platform
- **Apache Struts 7** MVC framework with Tiles layout management
- **Stateless EJB** session beans for business logic
- **Remote EJB** interface for distributed calls
- **JPA 3.1** with Hibernate and DB2
- **JDBC alternative** implementation with repository pattern
- **Maven multi-module** project structure
- **EAR packaging** for enterprise deployment

## Prerequisites

- **Java 17** or higher
- **Maven 3.8+** (or use the included Maven wrapper)
- **DB2** or "**PostgreSQL** Database
- **TomEE 10.1+ Plus**

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

Note: you need to put a Violation.properties into your current folder or into your user's home folder:

```bash
cp VioApp/Properties/Violation_lam01demo-webapp-01.properties ~/Violation.properties
```

## Deployment Instructions

### Apache TomEE 10.0+ Plus

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

### Debugging

Enable detailed logging:

**TomEE**: Edit `conf/logging.properties`
```properties
.level = INFO
com.example.jakartaee.level = FINE
org.apache.struts2.level = FINE
```

## Additional Resources

- [JakartaEE Documentation](https://jakarta.ee/)
- [Apache Struts Documentation](https://struts.apache.org/)
- [TomEE Documentation](https://tomee.apache.org/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Awesome Jakarta EE Resources for Developers](https://awesome-jakartaee.github.io/)

---

## Using AI in development

### SpecKit

Spec-Driven Development flips the script on traditional software development. For decades, code has been king — specifications were just scaffolding we built and discarded once the "real work" of coding began. Spec-Driven Development changes this: specifications become executable, directly generating working implementations rather than just guiding them.

Note: the GitHub SpecKit works with any Cli AI tools (Claude Code, Gemini CLI, OpenAI Codex etc.)

#### Installation

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

### Claude Code

#### Installation

```bash
$ curl -fsSL https://claude.ai/install.sh | bash

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

### Claude Desktop

Windows/MacOS: https://claude.com/download

Linux;
https://github.com/aaddrick/claude-desktop-debian/releases
https://github.com/aaddrick/claude-desktop-debian/releases/download/v1.1.10%2Bclaude0.14.10/claude-desktop-0.14.10-amd64.AppImage

## Git-SVN

1. Optional. Clone Workspace repository (git):

```bash
git clone git@github.com:dmorozov/rctc_workspace.git
```

This repository contains .devcontainer, .vscode, and .claude configs

2. Clone RCTC SVN tracked repositories with git-svn:
```bash
cd rctc_workspace

git svn clone svn://10.0.50.46/RCTC/branches/rctc_v20/VioApp
git svn clone svn://10.0.50.46/RCTC/branches/rctc_v20/VioUtil
git svn clone svn://10.0.50.46/RCTC/branches/rctc_v20/VioEJB
git svn clone svn://10.0.50.46/RCTC/branches/rctc_v20/VioWeb
git svn clone svn://10.0.50.46/RCTC/branches/rctc_v20/VioWebAdmin
git svn clone svn://10.0.50.46/RCTC/branches/rctc_v20/VTX-TXAPI
git svn clone svn://10.0.50.46/RCTC/branches/rctc_v20/VioBatchClient
git svn clone svn://10.0.50.46/RCTC/branches/rctc_v20/VioLetterClient
git svn clone svn://10.0.50.46/RCTC/branches/rctc_v20/VioTaskClient

3. Git-SVN cheat list

| **Action**             | **Standard Git** | **git svn**       |
| ---------------------- | ---------------- | ----------------- |
| **Download project**   | `git clone`      | `git svn clone`   |
| **Get latest updates** | `git pull`       | `git svn rebase`  |
| **Share your work**    | `git push`       | `git svn dcommit` |
| **View history**       | `git log`        | `git svn log`     |

all other Git commands are the same: branching, stashing, etc
