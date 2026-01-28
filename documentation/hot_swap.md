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

