---
type: Note
belongs_to: "[[aixle-flow]]"
---
# Rodrigo - Billy 09/03/26

## Aixle Flow

The POC was to automate the code review with a set of orchestrated agents that perform the assessment from different optics.

1. Manage securely client source code
2. Download NVD DB → Run vulnerabilities assessment
3. Create ephimeral SonarQube → Run assessment
4. Call 5 agents to answer the questionaire → 5 Assessments
5. Consolidate all that in our templated report

### Some challenges

- Access to ECR
- Ephimeral environments
- Custom software needed for specific tasks
- Carry over work from previous workflows and sessions

### Two different approaches

1. Multi-staged board and workflows
2. Multi-session workflow

### Gain points

- The MCP - It helped a lot to configure board, tasks, worflows, sessions. Without it, it would be way more challenging than it is.
- Fixes - How responsive is Artem and team about the errors I've found and the recommendations I made about the UI.
- Secrets and assets managemt - It was resolved really well and there was never an issuer with that. Perhaps having a MCP tool to update/read them would be nice.

### Pain points

- Documentation - Still no documentation, either internal or public. We should push for something that our team can use.
- Learning curve - Being the MCP the most efficient way to use Aixle, it makes even worse learning how to use the tool efficiently.
- Never-ending sessions - Even calling the finish\_session.rb tool, session didn't gracefully ended, making then to take over 30 minutes to complete.
- Cheatsheet - There are too many hiden features, MCP, Ruby tool, simultaneous sessions that are not documented or available to the end-user.
- HW limitations - For some tasks I was limited by the max amount of memory that the Pod has. There is no recommendation or remediation for that.

## Infios

What we know

- Full Micrsoft (ASP.NET Web Forms, C# and C++)
- What is C++? It's used for low-level or performance-sensitive components 
- VmWare to EC2
- MSSQL server with tons of SP and undocumented code in them

## AWS Transform

1. VmWare to EC2 → [https://aws.amazon.com/transform/vmware/](https://aws.amazon.com/transform/vmware/)
2. 

##
