# Amazon Bedrock Chat POC

## Overview of Solution



![A gif of a screen recording show casing the Amazon Bedrock Chat POC functionality](images/demo.gif)


## Goal of this POC
The goal of this repo is to provide users the ability to use Amazon Bedrock in a similar fashion to ChatGPT. This repo comes with a basic frontend to help users stand up a proof of concept in just a few minutes.

The architecture & flow of the POC is as follows:
![POC Architecture & Flow](images/architecture.png 'POC Architecture')


When a user interacts with the POC, the flow is as follows:

1. The user makes a &quot;zero-shot&quot; request to the streamlit frontend (`app.py`)

1. The application returns the 3 most semantically similar prompts, and creates a final prompt that contains the 3 returned prompts along with users query (few-shot prompting) (`prompt_finder_and_invoke_llm.py`).

1. The final prompt is passed into Amazon Bedrock to generate an answer to the users question (`prompt_finder_and_invoke_llm.py`).

1. The final answer is generated by Amazon Bedrock and displayed on the frontend application (`app.py`)




# How to use this Repo:

## Prerequisites:

1. [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) installed and configured with access to Amazon Bedrock.

1. [Python](https://www.python.org/downloads/) v3.11 or greater. The POC runs on python. 



## Steps
1. Clone the repository to your local machine.

    ```
    git clone https://github.com/aws-samples/genai-quickstart-pocs.git
    ```
    
    The file structure of this POC is broken into these files
    
    * `requirements.txt` - all the requirements needed to get the sample application up and running.
    * `app.py` - The streamlit frontend
    
    
    * `prompt_finder_and_invoke_llm.py` - houses the logic of the application, including the semantic search against the prompt repository and prompt formatting logic and the Amazon Bedrock API invocations.
    
    * `chat_history_prompt_generator.py` - houses the logic required to preserve session state and to dynamically inject the conversation history into prompts to allow for follow-up questions and conversation summary.
    
    

1. Open the repository in your favorite code editor. In the terminal, navigate to the POC's folder:
    ```zsh
    cd genai-quickstart-pocs-python/amazon-bedrock-chat-poc
    ```

1. Configure the python virtual environment, activate it & install project dependencies. *Note: each POC has it's own dependencies & dependency management.*
    ```zsh
    python -m venv .env
    source .env/bin/activate
    pip install -r requirements.txt
    ```

1. Create a .env file in the root of this repo. Within the .env file you just created you will need to configure the .env to contain:

    ```zsh
    profile_name=<AWS_CLI_PROFILE_NAME>
    ```


1. Depending on the region and model that you are planning to use Amazon Bedrock in, you may need to reconfigure line 23 in the prompt_finder_and_invoke_llm.py file to set the appropriate region:

    ```zsh
    bedrock = boto3.client('bedrock-runtime', 'us-east-1', endpoint_url='https://bedrock-runtime.us-east-1.amazonaws.com')
    ```


1. Since this repository is configured to leverage Claude 3, the prompt payload is structured in a different format. If you wanted to leverage other Amazon Bedrock models you can replace the llm_answer_generator() function in the prompt_finder_and_invoke_llm.py to look like:

    ```zsh
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


1. Start the POC from your terminal
    ```zsh
    streamlit run app.py
    ```
This should start the POC and open a browser window to the application. 

## How-To Guide
For a details how-to guide for using this poc, visit [HOWTO.md](HOWTO.md)

