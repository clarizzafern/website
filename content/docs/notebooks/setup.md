+++
title = "Set Up Your Notebooks"
description = "Getting started with Jupyter notebooks on Kubeflow"
weight = 10
+++

Your Kubeflow deployment includes services for spawning and managing Jupyter
notebooks. 

You can set up multiple *notebook servers* per Kubeflow deployment. Each
notebook server can include multiple *notebooks*. Each notebook server belongs
to a single *namespace*, which corresponds to the project group or team for that
server.

This guide shows you how to set up a notebook server for your Jupyter notebooks
in Kubeflow.

## Quick guide

Summary of steps:

1. Follow the [Kubeflow getting-started guide](/docs/started/getting-started/) 
  to set up your Kubeflow deployment and open the Kubeflow UI.

1. Click **Notebook Servers** in the left-hand panel of the Kubeflow UI.
1. Choose the **namespace** corresponding to your Kubeflow profile.
1. Click **NEW SERVER** to create a notebook server.
1. When the notebook server provisioning is complete, click **CONNECT**.
1. Click **Upload** to upload an existing notebook, or click **New** to
  create an empty notebook.

The rest of this page contains details of the above steps.

## Install Kubeflow and open the Kubeflow UI

Follow the [Kubeflow getting-started guide](/docs/started/getting-started/) to
set up your Kubeflow deployment in your environment of choice (locally, on 
premises, or in the cloud).

When Kubeflow is running, access the Kubeflow UI as described in the
getting-started guide for your chosen environment. Follow [accessing Kubeflow UIs guide](/docs/other-guides/accessing-uis/) for instructions on how to connect to Kubeflow UI.

## Create a Jupyter notebook server and add a notebook

1. Click **Notebook Servers** in the left-hand panel of the Kubeflow UI to 
  access the Jupyter notebook services deployed with Kubeflow:
  <img src="/docs/images/jupyterlink.png" 
    alt="Opening notebooks from the Kubeflow UI"
    class="mt-3 mb-3 border border-info rounded">

1. Sign in:
   * On GCP, sign in using your Google Account. (If you have already logged in
     to your Google Account you may not need to log in again.)
   * On all other platforms, sign in using any username and password.

1. Select a namespace:
   * Click the namespace dropdown to see the list of available namespaces.
   * Choose the namespace that corresponds to your Kubeflow profile. (See
     the page on [multi-user isolation](/docs/other-guides/multi-user-overview/) 
     for more information about namespaces.)

    <img src="/docs/images/notebooks-namespace.png" 
      alt="Selecting a Kubeflow namespace"
      class="mt-3 mb-3 border border-info rounded">

1. Click **NEW SERVER** on the **Notebook Servers** page:

    <img src="/docs/images/add-notebook-server.png" 
      alt="The Kubeflow notebook servers page"
      class="mt-3 mb-3 border border-info rounded">

    You should see a page for entering details of your new server. Here is a 
    partial screenshot of the page:

    <img src="/docs/images/new-notebook-server.png" 
      alt="Form for adding a Kubeflow notebook server"
      class="mt-3 mb-3 border border-info rounded">

1. Enter a **name** of your choice for the notebook server. The name can
  include letters and numbers, but no spaces. For example, `my-first-notebook`.
1. Kubeflow automatically updates the value in the **namespace** field to
  be the same as the namespace that you selected in a previous step. This 
  ensures that the new notebook server is in a namespace that you can access.

1. Select a Docker **image** for the baseline deployment of your notebook 
  server. You can choose from a range of *standard* images or specify a 
  *custom* image:

  * **Standard**: The standard Docker images include typical machine learning 
    (ML) packages that you can use within your Jupyter notebooks on 
    this notebook server. Select an image from the **Image** dropdown menu.
    The image names indicate the following choices:

      * A TensorFlow version (for example, `tensorflow-1.13.1`). Kubeflow offers
        a CPU and a GPU image for each minor version of TensorFlow.
      * `cpu` or `gpu`, depending on whether you want to train your model on a CPU 
        or a GPU. 
        
          * If you choose a GPU image, make sure that you have GPUs 
            available in your Kubeflow cluster. Run the following command to check 
            if there are any GPUs available:
            `kubectl get nodes "-o=custom-columns=NAME:.metadata.name,GPU:.status.allocatable.nvidia\.com/gpu"`
          * If you have GPUs available, you can schedule your server on a GPU node 
            in the **Extra Resources** section at the bottom of the form. For 
            example, to reserve two GPUs, enter the following JSON code: 
          `{"nvidia.com/gpu": 2}`
      * Kubeflow version (for example, `v0.5.0`).
    

  * **Custom**: If you select the custom option, you must specify a Docker image 
    in  the form `registry/image:tag`. For guidelines on creating a Docker
    image for your notebook, see the guide to 
    [creating a custom Jupyter image](/docs/notebooks/custom-notebook/).

1. Specify the total amount of **CPU** that your notebook server should reserve. 
  The default is `0.5`. For CPU-intensive jobs, you can choose more than one CPU 
  (for example, `1.5`).

1. Specify the total amount of **memory** (RAM) that your notebook server should 
  reserve. The default is `1.0Gi`.

1. Specify a **workspace volume** to hold your personal workspace for this
  notebook server. Kubeflow provisions a 
  [Kubernetes persistent volume (PV)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) for your workspace volume. The PV ensures that you can
  retain data even if you destroy your notebook server.

  * The default is to create a new volume for your workspace with the
    following configuration:
  
      * Name: The volume name is synced with the name of the notebook server.
        When you start typing the notebook server name, the volume name takes 
        the same value. You can edit the volume name, but if you later edit the 
        notebook server name, the volume name changes to match the notebook 
        server name.
      * Size: `10Gi`
      * Mount path: `/home/jovyan`
      * Access mode: `ReadWriteOnce`. This setting means that the volume can be 
        mounted as read-write by a single node. See the 
        [Kubernetes documentation](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) for more details about access modes.

  * Alternatively, you can point the notebook server at an existing volume by 
    specifying the name, mount path, and access mode for the existing volume.

1. *(Optional)* Specify one or more **data volumes** if you want to store and
  access data from the notebooks on this notebook server. You can add new
  volumes or specify existing volumes. Kubeflow provisions a 
  [Kubernetes persistent volume (PV)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) for each of your data volumes.

1. Click **LAUNCH**. You should see an entry for your new
  notebook server on the **Notebook Servers** page, with a spinning indicator in 
  the **Status** column. It can take a few minutes to set up
  the notebook server.

    * You can check the status of your Pod by hovering your mouse cursor over 
      the icon in the **Status** column next to the entry for your notebook
      server. For example, if the image is downloading then the status spinner 
      has a tooltip that says `ContainerCreating`.

        Alternatively, you can check the Pod status by entering the following 
        command:

        ```
        kubectl -n <NAMESPACE> describe pods jupyter-<USERNAME>
        ```

        Where `<NAMESPACE>` is the namespace you specified earlier 
        (default `kubeflow`) and `<USERNAME>` is the name you used to log in.
        **A note for GCP users:** If you have IAP turned on, the Pod has
        a different name. For example, if you signed in as `USER@DOMAIN.EXT` 
        the Pod has a name of the following form:

        ```
        jupyter-accounts-2egoogle-2ecom-3USER-40DOMAIN-2eEXT
        ```

1. When the notebook server provisioning is complete, you should see an entry
  for your server on the **Notebook Servers** page, with a check mark in the
  **Status** column:

    <img src="/docs/images/notebook-servers.png" 
      alt="Opening notebooks from the Kubeflow UI"
      class="mt-3 mb-3 border border-info rounded">

1. Click **CONNECT** to start the notebook server.

1. When the notebook server is running, you should see the Jupyter dashboard
  interface. If you requested a new workspace, the dashboard should be empty
  of notebooks:

    <img src="/docs/images/jupyter-dashboard.png" 
      alt="Jupyter dashboard with no notebooks"
      class="mt-3 mb-3 border border-info rounded">

1. Click **Upload** to upload an existing notebook, or click **New** to
  create an empty notebook. You can read about using notebooks in the
  [Jupyter documentation](https://jupyter-notebook.readthedocs.io/en/latest/notebook.html#notebook-user-interface).

## Experiment with your notebook

The default notebook image includes all the plugins that you need to train a 
TensorFlow model with Jupyter, including
[Tensorboard](https://www.tensorflow.org/get_started/summaries_and_tensorboard) 
for rich visualizations and insights into your model.

To test your Jupyter installation, you can run a basic 'hello world' program
(adapted from
[mnist_softmax.py](https://github.com/tensorflow/tensorflow/blob/r1.4/tensorflow/examples/tutorials/mnist/mnist_softmax.py)) as follows:

1. Use the Jupyter dashboard to create a new **Python 3** notebook.

1. Copy the following code and paste it into a code block in your notebook:

    ```
    from tensorflow.examples.tutorials.mnist import input_data
    mnist = input_data.read_data_sets("MNIST_data/", one_hot=True)

    import tensorflow as tf

    x = tf.placeholder(tf.float32, [None, 784])

    W = tf.Variable(tf.zeros([784, 10]))
    b = tf.Variable(tf.zeros([10]))

    y = tf.nn.softmax(tf.matmul(x, W) + b)

    y_ = tf.placeholder(tf.float32, [None, 10])
    cross_entropy = tf.reduce_mean(-tf.reduce_sum(y_ * tf.log(y), reduction_indices=[1]))

    train_step = tf.train.GradientDescentOptimizer(0.05).minimize(cross_entropy)

    sess = tf.InteractiveSession()
    tf.global_variables_initializer().run()

    for _ in range(1000):
      batch_xs, batch_ys = mnist.train.next_batch(100)
      sess.run(train_step, feed_dict={x: batch_xs, y_: batch_ys})

    correct_prediction = tf.equal(tf.argmax(y,1), tf.argmax(y_,1))
    accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
    print("Accuracy: ", sess.run(accuracy, feed_dict={x: mnist.test.images, y_: mnist.test.labels}))
    ```

1. Run the code. You should see a number of `WARNING` messages from TensorFlow,
  followed by a line showing a training accuracy something like this:

    ```
    Accuracy:  0.9012
    ```

Please note that when running on most cloud providers, the public IP address is 
exposed to the internet and is an unsecured endpoint by default.

## Next steps

* See a [simple example](https://github.com/kubeflow/examples/tree/master/pipelines/simple-notebook-pipeline) of creating Kubeflow pipelines in a Jupyter notebook on GCP.
* Build machine-learning pipelines with the [Kubeflow Pipelines 
  SDK](/docs/pipelines/sdk/sdk-overview/).
* Explore [Kubeflow Fairing](/docs/fairing/) for a complete solution to 
  building, training, and deploying an ML model from a notebook.
* See how to configure [multi-user isolation](/docs/other-guides/multi-user-overview/) in Kubeflow, to separate the notebooks for each user in a shared Kubeflow deployment.
* Learn the advanced features available from a Kubeflow notebook, such as
  [submitting Kubernetes resources](/docs/notebooks/submit-kubernetes/) or
  [building Docker images](/docs/notebooks/submit-docker-image/). 
* Visit the [troubleshooting guide](/docs/notebooks/troubleshoot) for fixing common
  errors in creating Jupyter notebooks in Kubeflow

