# Using the Client

Besides the command line, LlamaPi provides a Windows client. The client can perform all operations covered in this guide: downloading, loading, running, and deploying models, and chatting with them in the client.

## DirectAI Client

The LlamaPi client app runs on the DirectAI platform. Download the DirectAI installer from the [Firefly website](https://community.t-firefly.com/doc/download/429), install it by following the [DirectAI documentation](https://community.t-firefly.com/docs/ai/applications/DirectAI/directai/introduction), and then open the client. DirectAI automatically scans the LAN for devices. Click a device to connect:

![client-usage-1](./images/client-usage/client-usage-1​.png)

## Chat with a Model

On the new-chat page, select a model to chat with. LlamaPi downloads and loads the model automatically:

![client-usage-0](./images/client-usage/client-usage-0.png)

Once the model is ready, inference runs and the result is returned:

![client-usage-00](./images/client-usage/client-usage-00.png)

During the conversation, you can customize the model's parameter settings:

![client-usage-000](./images/client-usage/client-usage-000.png)

## Manage Local Models

On the LlamaPi `Model Library` page, browse the models available on the current hardware, download the ones you need, and delete the ones you no longer need:

![client-usage-2](./images/client-usage/cli​ent-usage-2.png)

## Load and Deploy Models

![client-usage-03](./images/client-usage/cli​ent-usage-03.png)

After a model is downloaded, open the `Deployment Config` page and click `New Deployment` to create a deployment configuration from a downloaded model. You can customize the deployment ID and the number of model instances, and configure the model's generation parameters:

![client-usage-3](./images/client-usage/client-usage-3.png)

A saved deployment configuration can be loaded and unloaded on demand, or configured to load automatically when the device starts:

![client-usage-4](./images/client-usage/client-usage-4.png)

## Chat with a Model Deployment

On the LlamaPi `New Chat` page, select a loaded model group to chat with:

![client-usage-5](./images/client-usage/client-usage-5.png)

Send messages to the running model deployment:

![client-usage-6](./images/client-usage/client-usage-6.png)
