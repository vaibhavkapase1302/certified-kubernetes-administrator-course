# Provisioning Compute Resources

We will provision the following infrastructure. The infrastructure will be created by Terraform, so as not to spend too much of the lab time just getting that provisioned, and to allow you to focus on the cluster installation.

![Infra](../../../images/kubeadm-aws-architecture.png)


As can be seen in this diagram, we will create three EC2 instances to form the cluster and a further one `student-node` from which to perform the configuration. We build the infrastructure using Terraform from AWS CloudShell (so you don't have to install Terraform on your workstation), then log into `student-node` which can access the cluster nodes. This relationship between `student-node` and the cluster nodes is similar to CKA Ultimate Mocks and how the real exam works - you start on a separate node (in this case `student-node`), then use SSH to connect to cluster nodes. Note that SSH connections are only possible in the direction of the arrows. It is not possible to SSH from e.g. `controlplane` directly to `node01`. You must `exit` to `student-node` first. This is also how it is in the exam. `student-node` assumes the role of a [bastion host](https://en.wikipedia.org/wiki/Bastion_host).

We will also set up direct connection from your workstation to the node ports of the workers so that you can browse any NodePort services you create (see security below).

Some basic security will be configured:

* Only the `student-node` will be able to access the cluster's API Server, and this is where you will run `kubectl` commands from when the cluster is running.
* Only the `student-node` can SSH to the cluster nodes.
* Ports required by Kubernetes itself (inc. etcd) and Weave CNI will be configured in security groups on the cluster nodes.

Security issues that would make this unsuitable for a genuine production cluster:

* The kube nodes should be on private subnets (no direct access from the Internet) and placed behind a NAT gateway to allow them to download packages, or with a more extreme security posture, completely [airgapped](https://en.wikipedia.org/wiki/Air_gap_(networking)).
* Access to API server and etcd would be more tightly controlled.
* Use of default VPC is not recommended.
* The node ports will be open to the world - i.e. anyone can connect to them.
* A cloud load balancer coupled with an ingress controller would be provisioned to provide ingress to the cluster. It is _definitely_ not recommended to expose the worker nodes' node ports to the Internet as we are doing here!!!

Other things that will be configured by the Terraform code
* Host names set on the nodes: `controlplane`, `node01`, `node02`
* Content of `/etc/hosts` set up on all nodes for easy use of `ssh` command from `student-node`.
* Generation and distribution of a key pair for logging into instances via SSH.

Let's go ahead and get the infrastructure built!

[Click here](https://kodekloud.com/topic/playground-aws/) to start a playground, and click `START LAB` to request a new AWS Cloud Playground instance. After a few seconds, you will receive a URL and your credentials to access AWS Cloud console.

Note that you must have KodeKloud Pro subscription to run an AWS playground. If you have your own AWS account, this should still work, however you will bear the cost for any resources created until you delete them.

We will run this entire lab in AWS CloudShell which is a Linux terminal you run inside the AWS console and has most of what we need preconfigured, such as git and the AWS credentials needed by Terraform. [Click here](https://us-east-1.console.aws.amazon.com/`student-node`/home?region=us-east-1) to open CloudShell.


## Install Terraform

From the CloudShell command prompt...

```bash
curl -O https://releases.hashicorp.com/terraform/1.6.2/terraform_1.6.2_linux_amd64.zip
unzip terraform_1.6.2_linux_amd64.zip
mkdir -p ~/bin
mv terraform ~/bin/
terraform version
```

## Clone this repo

```bash
git clone https://github.com/kodekloudhub/certified-kubernetes-administrator-course.git
```

Now change into the `aws/terraform` directory

```bash
cd certified-kubernetes-administrator-course/kubeadm-clusters/aws/terraform
```

## Provision the infrastructure

1. Run the terraform

    ```bash
    terraform init
    terraform plan
    terraform apply
    ```

    This should take about half a minute. If this all runs correctly, you will see something like the following at the end of all the output. IP addresses _will be different_ for you

    ```
    Apply complete! Resources: 22 added, 0 changed, 0 destroyed.

    Outputs:

    address_node01 = "18.233.150.22"
    address_node02 = "54.87.18.1"
    address_student_node = "100.26.200.3"
    connect_student_node = "ssh ubuntu@100.26.200.3"
    ```

    Copy all these outputs to a notepad for later use.

1. Wait for all instances to be ready (Instance state - `running`, Status check - `2/2 checks passed`). This will take 2-3 minutes. See [EC2 console](https://us-east-1.console.aws.amazon.com/ec2/home?region=us-east-1#Instances:instanceState=running).

1. Log into `student-node`

    Copy the `ssh` command from the terraform output `connect_student_node`, e.g.

    ```
    ssh ubuntu@100.26.200.3
    ```

    Note that the IP address _will be different_ for you.

## Prepare the student node

We will install kubectl here so that we can run commands against the cluster when it is built

1. Install latest version of kubectl and place in the user programs directory
    ```bash
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    chmod +x kubectl
    sudo mv kubectl /usr/local/bin
    ```

1. Check

    ```bash
    kubectl version
    ```

    It should amongst other things tell you

    > The connection to the server localhost:8080 was refused - did you specify the right host or port?

    which is fine, since we haven't installed kubernetes yet.


Next: [Connectivity](./03-connectivity.md)<br/>
Prev: [Prerequisites](./01-prerequisites.md)