Azure Functions:
- Azure function is a server-less, event driven compute service.
- Azure Functions is a serverless computing service that allows developers to execute event-driven code without managing infrastructure.
- It simplifies application development by enabling you to focus on writing code while Azure handles the underlying infrastructure, scaling, and maintenance.
- This approach reduces costs and operational overhead.
- Without the Azure functions we have to use VM to run some function or code, this requires maintaining the infra and also more cost
- Example use case: Run a Azure function each time when there is a upload to blob storage, process the file and store it in a DB.
- Function App:
  - A Function App is the Azure resource that hosts one or more functions.
- So we create Function App and inside that we create functions.
- Functions can be creates in 3 ways: Portal, VisualStudio, CLI
- Trigger:
  - When to execute the function
  - It can be for EventHub trigger, http request, Blob uploaded, Queue trigger or run every 5 minutes etc
  - We can also create our own triggers
- Binding:
  - Connecting the function to another service,

App Service
- "I have an application that needs a managed web hosting platform."
- For application that runs continuously, may be web application, APIs

Functions
→ "I have pieces of code that should execute when something happens."
  
