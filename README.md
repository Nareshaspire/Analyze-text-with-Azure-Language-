# Analyze-text-with-Azure-Language-
Develop AI Language and Speech solutions on Azure.

Prerequisites
Create a Microsoft Foundry project
Get the application files from GitHub
Configure your application
Add code to connect to your Azure AI Language resource
Add code to detect language
Add code to extract entities
Add code to redact PII
Clean up
Analyze Text
Azure Language in Foundry Tools supports analysis of text, including language detection, entity recognition, and PII redaction.

For example, suppose a travel agency wants to process hotel reviews that have been submitted to the company's web site. By using the Azure Language, they can determine the language each review is written in, identify named entities, such as places, landmarks, or people mentioned in the reviews, and redact any personally identifiable information before publishing them on the company's website. In this exercise, you'll use the Azure Language Python SDK for text analytics to implement a simple hotel review application.

While this exercise is based on Python, you can develop text analytics applications using multiple language-specific SDKs; including:

The code used in this exercise is based on the for Microsoft Foundry Tools SDK for Python. You can develop similar solutions using the SDKs for Microsoft .NET, JavaScript, and Java. Refer to Microsoft Foundry SDK client libraries for details.

This exercise takes approximately 30 minutes.

Note: Some of the technologies used in this exercise are in preview or in active development. You may experience some unexpected behavior, warnings, or errors.

Prerequisites
Before starting this exercise, ensure you have:

An active Azure subscription
Visual Studio Code installed
Python version 3.13.xx installed*
Git installed and configured
Azure CLI installed
* Python 3.14 is available, but some dependencies are not yet compiled for that release. The lab has been successfully tested with Python 3.13.12.

Create a Microsoft Foundry project
Microsoft Foundry uses projects to organize models, resources, data, and other assets used to develop an AI solution.

In a web browser, open the Microsoft Foundry portal at https://ai.azure.com and sign in using your Azure credentials. Close any tips or quick start panes that are opened the first time you sign in, and if necessary use the Foundry logo at the top left to navigate to the home page.

If it is not already enabled, in the tool bar the top of the page, enable the New Foundry option. Then, if prompted, create a new project with a unique name; expanding the Advanced options area to specify the following settings for your project:

Foundry resource: Use the default name for your resource (usually {project_name}-resource)
Subscription: Your Azure subscription
Resource group: Create or select a resource group
Region: Select any available region
Select Create. Wait for your project to be created.

On the home page for your project, note that the API key, project endpoint, and OpenAI endpoint are displayed here.

TIP: You're going to need the project endpoint later!

Get the application files from GitHub
The initial application files you'll need to develop the review analysis application are provided in a GitHub repo.

Open Visual Studio Code.

Open the command palette (Ctrl+Shift+P) and use the Git:clone command to clone the https://github.com/microsoftlearning/mslearn-ai-language repo to a local folder (it doesn't matter which one). Then open it.

You may be prompted to confirm you trust the authors.

After the repo has been cloned, in the Explorer pane, navigate to the folder containing the application code files at /Labfiles/01-analyze-text/Python/text-analysis. The application files include:

reviews (a subfolder containing the review documents)
.env (the application configuration file)
requirements.txt (the Python package dependencies that need to be installed)
text-analysis.py (the code file for the application)
Configure your application
In Visual Studio Code, view the Extensions pane; and if it is not already installed, install the Python extension.

In the Command Palette, use the command python:select interpreter. Then select an existing environment if you have one, or create a new Venv environment based on your Python 3.13.x installation.

Tip: If you are prompted to install dependencies, you can install the ones in the requirements.txt file in the /Labfiles/01-analyze-text/Python/text-analysis folder; but it's OK if you don't - we'll install them later!

Tip: If you prefer to use the terminal, you can create your Venv environment with python -m venv labenv, then activate it with \labenv\Scripts\activate.

In the Explorer pane, right-click the text-analysis folder containing the application files, and select Open in integrated terminal (or open a terminal in the Terminal menu and navigate to the /Labfiles/01-analyze-text/Python/text-analysis folder.)

Note: Opening the terminal in Visual Studio Code will automatically activate the Python environment. You may need to enable running scripts on your system.

Ensure that the terminal is open in the text-analysis folder with the prefix (.venv) to indicate that the Python environment you created is active.

Install the Azure Language Text Analytics SDK and other required packages by running the following command:

code
pip install -r requirements.txt
In the Explorer pane, in the text-analysis folder, select the .env file to open it. Then update the configuration values to include the endpoint (up to the .com domain) for your Foundry project (copy these from the Foundry portal).

Important: Modify the pasted endpoint to remove the "/api/projects/{project_name}" suffix - the endpoint should be https://{your-foundry-resource-name}.services.ai.azure.com.

Save the modified configuration file.

Add code to connect to your Azure AI Language resource
In the Explorer pane, in the text-analysis folder, open the text-analysis.py file.

Review the existing code. You will add code to work with the Azure Language Text Analytics SDK.

Tip: As you add code to the code file, be sure to maintain the correct indentation.

At the top of the code file, under the existing namespace references, find the comment Import namespaces and add the following code to import the namespaces you will need to use the Text Analytics SDK:

code
# import namespaces
from azure.identity import DefaultAzureCredential
from azure.ai.textanalytics import TextAnalyticsClient
In the main function, note that code to load the endpoint from the configuration file has already been provided. Then find the comment Create client using endpoint, and add the following code to create a client for the Text Analysis API:

code
# Create client using endpoint
credential = DefaultAzureCredential()
ai_client = TextAnalyticsClient(endpoint=foundry_endpoint, credential=credential)
Save the changes to the code file. Then, in the terminal pane, use the following command to sign into Azure.

code
az login
Note: In most scenarios, just using az login will be sufficient. However, if you have subscriptions in multiple tenants, you may need to specify the tenant by using the --tenant parameter. See Sign into Azure interactively using the Azure CLI for details.

When prompted, follow the instructions to sign into Azure. Then complete the sign in process in the command line, viewing (and confirming if necessary) the details of the subscription containing your Foundry resource.

After you have signed in, enter the following command to run the application:

code
python text-analysis.py
Observe the output as the code should run without error, displaying the contents of each review text file in the reviews folder. The application successfully creates a client for the Text Analytics API but doesn't make use of it. We'll fix that in the next section.

Add code to detect language
Now that you have created a client for the API, let's use it to detect the language in which each review is written.

In the code editor, find the comment Get language. Then add the code necessary to detect the language in each review document:

code
# Get language
detectedLanguage = ai_client.detect_language(documents=[text])[0]
print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
Note: In this example, each review is analyzed individually, resulting in a separate call to the service for each file. An alternative approach is to create a collection of documents and pass them to the service in a single call. In both approaches, the response from the service consists of a collection of documents; which is why in the Python code above, the index of the first (and only) document in the response ([0]) is specified.

Save your changes. Then re-run the program.

Observe the output, noting that this time the language for each review is identified.

Add code to extract entities
Often, documents or other bodies of text mention people, places, time periods, or other entities. The text Analytics API can detect multiple categories (and subcategories) of entity in your text.

In the code editor, find the comment Get entities. Then, add the code necessary to identify entities that are mentioned in each review:

code
# Get entities
entities = ai_client.recognize_entities(documents=[text])[0].entities
if len(entities) > 0:
    print("\nEntities")
    for entity in entities:
        print('\t{} ({})'.format(entity.text, entity.category))
Save your changes and re-run the program.

Observe the output, noting the entities that have been detected in the text.

Add code to redact PII
Often, privacy policies and legislation can require that personally identifiable information (PII). such as names, addresses, phone numbers, and other private details be redacted from documents.

In the code editor, find the comment Get PII. Then, add the code necessary to identify PII entities that are mentioned in each review:

code
# Get PII
pii_result = ai_client.recognize_pii_entities(documents=[text])[0]
pii_entities = pii_result.entities
if len(pii_entities) > 0:
    print("\nPII Entities")
    for pii_entity in pii_entities:
        print('\t{} ({})'.format(pii_entity.text, pii_entity.category)) 
    print("Redacted Text:\n {}".format(pii_result.redacted_text))
Save your changes and re-run the program.

Observe the output, noting the PII entities that are identified, and reviewing the redacted version of each document that is produced.

Clean up
If you've finished exploring Azure Language in Foundry Tools, you should delete the resources you have created in this exercise to avoid incurring unnecessary Azure costs.

Open the Azure portal and view the contents of the resource group where you deployed the resources used in this exercise.
On the toolbar, select Delete resource group.
Enter the resource group name and confirm that you want to delete it.
