# Amazon-Bedrock-Speech-to-Text-Chat-POC

This is sample code demonstrating the use of Amazon Bedrock and Generative AI to implement a ChatGPT alternative using speech-to-text prompts. The application is constructed with a simple streamlit frontend where users can provide zero shot requests using their computer’s microphone and listen to responses to satisfy a broad range of use cases.

![Alt text](images/demo.gif)
# **Goal of this Repo:**

The goal of this repo is to provide users the ability to use Amazon Bedrock in a similar fashion to ChatGPT by invoking the model using their speech rather than typing. This repo comes with a basic frontend to help users stand up a proof of concept in just a few minutes.

The architecture and flow of the sample application will be:

![Alt text](images/architecture.png "POC Architecture")

When a user interacts with the GenAI app, the flow is as follows:

1. The user makes a "zero-shot" request to the streamlit frontend (app.py) by speaking into their computer’s microphone.
2. The live audio is streamed to Amazon Transcribe (live_transcription.py)
3. The transcribed text is streamed back to the streamlit frontend (app.py) once the user has finished speaking.
4. The application performs a semantic search of the users query against the 1200+ prompts. (prompt_finder_and_invoke_llm.py).
5. The application returns the 3 most semantically similar prompts, and creates a final prompt that contains the 3 returned prompts along with users query (few-shot prompting) (prompt_finder_and_invoke_llm.py).
6. The final prompt is passed into Amazon Bedrock to generate an answer to the user’s question (prompt_finder_and_invoke_llm.py).
7. The final answer is generated by Amazon Bedrock and displayed on the frontend application (app.py).
8. The final answer is passed to Amazon Polly to convert the text to natural sounding speech (app.py)
9. The audio file is returned and played through the frontend application (app.py).

# How to use this Repo:

## Prerequisites:

1. Amazon Bedrock Access and CLI Credentials. Ensure that the proper FM model access is provided in the Amazon Bedrock console
2. Ensure Python 3.10 installed on your machine, it is the most stable version of Python for the packages we will be using, it can be downloaded [here](https://www.python.org/downloads/release/python-3911/).

## Step 1:

The first step of utilizing this repo is performing a git clone of the repository.

```
git clone https://github.com/aws-samples/genai-quickstart-pocs.git
```

After cloning the repo onto your local machine, open it up in your favorite code editor. The file structure of this repo is broken into 5 key files, the app.py file, the prompt_finder_and_invoke_llm.py file, the chat_history_prompt_generator.py file, the live_transcription.py file, and the requirements.txt. The app.py file houses the frontend application (a streamlit app). The prompt_finder_and_invoke_llm.py file houses the logic of the application, including the semantic search against the prompt repository and prompt formatting logic and the Amazon Bedrock API invocations. The chat_history_prompt_generator.py houses the logic required to preserve session state and to dynamically inject the conversation history into prompts to allow for follow-up questions and conversation summary. The live_transcription.py file house the logic required to create an audio stream from the users microphone, send the audio chunks to Amazon Transcribe, and generate a text transcript. The requirements.txt file contains all necessary dependencies for this sample application to work.

## Step 2:

Set up a python virtual environment in the root directory of the repository and ensure that you are using Python 3.9. This can be done by running the following commands:

```
pip install virtualenv
python3.10 -m venv venv
```

The virtual environment will be extremely useful when you begin installing the requirements. If you need more clarification on the creation of the virtual environment please refer to this [blog](https://www.freecodecamp.org/news/how-to-setup-virtual-environments-in-python/).
After the virtual environment is created, ensure that it is activated, following the activation steps of the virtual environment tool you are using. Likely:

```
cd venv
cd bin
source activate
cd ../../
```

After your virtual environment has been created and activated, you can install all the requirements found in the requirements.txt file by running this command in the root of this repos directory in your terminal:

```
pip install -r requirements.txt
```

## Step 3:

Now that the requirements have been successfully installed in your virtual environment we can begin configuring environment variables.
You will first need to create a .env file in the root of this repo. Within the .env file you just created you will need to configure the .env to contain:

```
profile_name=<AWS_CLI_PROFILE_NAME>
```

Please ensure that your AWS CLI Profile has access to Amazon Bedrock!

Depending on the region and model that you are planning to use Amazon Bedrock in, you may need to reconfigure line 23 in the prompt_finder_and_invoke_llm.py file to set the appropriate region:

```
bedrock = boto3.client('bedrock-runtime', 'us-east-1', endpoint_url='https://bedrock-runtime.us-east-1.amazonaws.com')
```

Since this repository is configured to leverage Claude 3 Haiku, the prompt payload is structured in a different format. If you wanted to leverage other Amazon Bedrock models you can replace the llm_answer_generator() function in the prompt_finder_and_invoke_llm.py to look like:

```python
def llm_answer_generator(question_with_prompt):
    """
    This function is used to invoke Amazon Bedrock using the finalized prompt that was created by the prompt_finder(question)
    function.
    :param question_with_prompt: This is the finalized prompt that includes semantically similar prompts, chat history,
    and the users question all in a proper multi-shot format.
    :return: The final answer to the users question.
    """
    # body of data with parameters that is passed into the bedrock invoke model request
    # TODO: TUNE THESE PARAMETERS AS YOU SEE FIT
    body = json.dumps({"prompt": question_with_prompt,
                       "max_tokens_to_sample": 8191,
                       "temperature": 0,
                       "top_k": 250,
                       "top_p": 0.5,
                       "stop_sequences": []
                       })
    # configure model specifics such as specific model
    modelId = 'anthropic.claude-v2'
    accept = 'application/json'
    contentType = 'application/json'
    # Invoking the bedrock model with your specifications
    response = bedrock.invoke_model(body=body,
                                    modelId=modelId,
                                    accept=accept,
                                    contentType=contentType)
    # the body of the response that was generated
    response_body = json.loads(response.get('body').read())
    # retrieving the specific completion field, where you answer will be
    answer = response_body.get('completion')
    # returning the answer as a final result, which ultimately gets returned to the end user
    return answer
```
You can then change the modelId variable to the model of your choice.

## Step 4:

As soon as you have successfully cloned the repo, created a virtual environment, activated it, installed the requirements.txt, and created a .env file, your application should be ready to go.
To start up the application with its basic frontend you simply need to run the following command in your terminal while in the root of the repositories' directory:

```
streamlit run app.py
```

As soon as the application is up and running in your browser of choice you can begin asking zero-shot questions using your computer’s microphone and leveraging this app as you would ChatGPT.
