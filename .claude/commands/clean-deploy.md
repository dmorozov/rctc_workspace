---
description: Clean up TomEE server deployment by removing deployed applications and work directories
allowed-tools: Bash(rm -rf /opt/tomcat/webapps/*), Bash(rm -rf /opt/tomcat/work/*), Bash(ls:*)
---

# Clean Server Deployment

## Target paths to clean

Webapps:
- /opt/tomcat/webapps/Violation
- /opt/tomcat/webapps/VioWeb
- /opt/tomcat/webapps/VioAdmin
- /opt/tomcat/webapps/VioWebAdmin
- /opt/tomcat/webapps/tx
- /opt/tomcat/webapps/VTX-TXAPI
- /opt/tomcat/webapps/rctc
- /opt/tomcat/webapps/rctc.ear

Work directories:
- /opt/tomcat/work/Catalina/localhost/Violation
- /opt/tomcat/work/Catalina/localhost/VioWeb
- /opt/tomcat/work/Catalina/localhost/VioAdmin
- /opt/tomcat/work/Catalina/localhost/VioWebAdmin
- /opt/tomcat/work/Catalina/localhost/tx
- /opt/tomcat/work/Catalina/localhost/VTX-TXAPI
- /opt/tomcat/work/Catalina/localhost/rctc

## Instructions

1. First, run `ls /opt/tomcat/webapps/` and `ls /opt/tomcat/work/Catalina/localhost/` to check which target paths exist
2. For paths that exist, delete them using direct `rm -rf` commands (NOT in a loop). Use separate commands like:
   - `rm -rf /opt/tomcat/webapps/Violation /opt/tomcat/webapps/VioAdmin /opt/tomcat/webapps/tx`
   - `rm -rf /opt/tomcat/work/Catalina/localhost/Violation /opt/tomcat/work/Catalina/localhost/VioAdmin`
3. Do NOT use for loops or shell scripting - use direct rm -rf commands with space-separated paths

## Output rules

- If ANY paths were deleted: Show "**Deployment cleaned up successfully.**" followed by a list of deleted paths
- If NO paths were deleted (none of the defined paths existed): Show "**The deployment was already clean.**"
- Do NOT show any error messages for paths that don't exist
