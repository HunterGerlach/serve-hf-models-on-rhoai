# webinar-transforming-aiml-concepts-to-reality

Deploying a fine-tuned model from HuggingFace

- In HuggingFace.co
  - Create an account if you don't already have one
  - Obtain a token: https://huggingface.co/settings/tokens
  - Optionally, you can create a secret directly in OpenShift to house the token via the following:

```yaml
kind: Secret
apiVersion: v1
metadata:
  name: huggingface-secret
  namespace: webinar-transforming-aiml-concepts-to-reality
type: Opaque
stringData:
  token: <token-goes-here>
```

- In OpenShift
  - Create a Minio instance by following: https://ai-on-openshift.io/tools-and-applications/minio/minio/#deploy-minio-in-your-project
  - Get the route
  - Get the secret: `minio`:`minio123`
  - Login to Minio
  - Create a bucket: `my-ai-bucket`
- In RHOAI
  - Create a data science project: `webinar-transforming-aiml-concepts-to-reality`
  - Create a workbench:
    - Name: `llama-finetune`
    - Image Selection: `PyTorch`
    - Version selection: `2023.2 (Recommended)`
    - Container size: `Medium`
    - Accelerator: `NVIDIA GPU`
    - Number of accelerators: `1`
    - ![alt text](img/image.png)
  - Add an environment variable for your HuggingFace token:
    - Select `Secret` and then choose `Key/value`
    - Enter a Key: `hf_token`
    - Enter the value obtained from your HuggingFace account
    - Note: when you're in your Jupyter Notebook, you can use the `%env hf_token` to see the value of the environment variable and ensure it is indeed set correctly.
    - ![alt text](img/image-1.png)
  - Create new persistent storage: `true` (selected)
    - Name: `llama-finetune`
    - Persistent storage size: `100 Gi` (this size depends on how big the models are that you'll be pulling down; note: the default 20Gi is not likely enough.)
    - ![alt text](img/image-2.png)
  - Use data connection: `true` (selected)
    - Create new data connection: `true` (selected)
    - Name: `webinar-data-connection`
    - Access Key: `minio` (default user)
    - Secret Key: `minio123` (default user pw)
    - Endpoint: API route (NOT the UI route)
    - Region: `us-east-1` (value does not actually matter)
    - Bucket: `my-ai-bucket`
    - ![alt text](img/image-3.png)
  - Select "Create workbench" - this may take some time
- Once started, enter into the workbench
- Create a new Jupyter Notebook or `git clone` this repo
  - If creating a new ipynb file, add each line from the repo's ipynb
  - Step through each line via SHIFT+ENTER, waiting for each step to fully complete before proceeding to the next one (`*` means the cell is still progressing)
  - Once all cells have completed, you should have a local version of the model you pulled from HuggingFace that is converted and ready to use
- Deploy the model:
  - Go back to the Data Science Project page
  - Select Single-model serving platform's "Deploy model" option
  - Enter the Deploy model section
  - Specify the Model name: `Llama-2-7b-chat-hf-fine-tuned`
  - Serving Runtime: `Caikit TGIS ServingRuntime for KServe`
  - Model framework: `caikit`
  - Model server replicas: `1`
  - Compute resources per replica: `Medium`
  - Accelerator: `NVIDIA GPU`
  - Model Location:
    - Existing data connection: `true` (selected)
    - Name: `webinar-data-connection`
    - Path: `model`
  - Select Deploy
    - This will use the data connection to pull the model from the Minio S3 bucket and deploy it onto OpenShift via RHOIA's Single Model Serving capability
