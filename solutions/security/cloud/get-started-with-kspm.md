---
mapped_pages:
  - https://www.elastic.co/guide/en/security/current/get-started-with-kspm.html
  - https://www.elastic.co/guide/en/serverless/current/security-get-started-with-kspm.html
applies_to:
  stack: all
  serverless:
    security: all
products:
  - id: security
  - id: cloud-serverless
---

# Get started with KSPM

This page explains how to configure the Kubernetes Security Posture Management (KSPM) integration.

::::{admonition} Requirements
* The KSPM integration is available to all Elastic Cloud users. For on-prem deployments, it requires an [Enterprise subscription](https://www.elastic.co/pricing).
* The KSPM integration only works in the `Default` Kibana space. Installing the KSPM integration on a different Kibana space will not work.
* KSPM is not supported on EKS clusters in AWS GovCloud. [Click here to request support](https://github.com/elastic/kibana/issues/new/choose).
* To view posture data, ensure you have the `read` privilege for the following {{es}} indices:

    * `logs-cloud_security_posture.findings_latest-*`
    * `logs-cloud_security_posture.scores-*`
    * `logs-cloud_security_posture.findings`


::::


The instructions differ depending on whether you’re installing on EKS or on unmanaged clusters.

* Install on EKS-managed clusters:

    1. [Name your integration and select a Kubernetes deployment type](/solutions/security/cloud/get-started-with-kspm.md#kspm-setup-eks-start)
    2. [Authenticate to AWS](/solutions/security/cloud/get-started-with-kspm.md#kspm-setup-eks-auth)
    3. [Finish configuring the KSPM integration](/solutions/security/cloud/get-started-with-kspm.md#kspm-setup-eks-finish)
    4. [Deploy the DaemonSet to your clusters](/solutions/security/cloud/get-started-with-kspm.md#kspm-setup-eks-modify-deploy)

* Install on unmanaged clusters:

    1. [Configure the KSPM integration](/solutions/security/cloud/get-started-with-kspm.md#kspm-setup-unmanaged)
    2. [Deploy the DaemonSet manifest to your clusters](/solutions/security/cloud/get-started-with-kspm.md#kspm-setup-unmanaged-modify-deploy)



## Set up KSPM for Amazon EKS clusters [kspm-setup-eks-start]


### Name your integration and select a Kubernetes Deployment type [_name_your_integration_and_select_a_kubernetes_deployment_type]

1. Find **Cloud Security Posture** in the navigation menu or use the [global search field](/explore-analyze/find-and-organize/find-apps-and-objects.md).
2. Click **Add a KSPM integration**.
3. Read the integration’s description to understand how it works. Then, click [**Add Kubernetes Security Posture Management**](https://docs.elastic.co/en/integrations/cloud_security_posture).
4. Name your integration. Use a name that matches the purpose or team of the cluster(s) you want to monitor, for example, `IT-dev-k8s-clusters`.
5. Select **EKS** from the **Kubernetes Deployment** menu. A new section for AWS credentials will appear.


### Authenticate to AWS [kspm-setup-eks-auth]

There are several options for how to provide AWS credentials:

* [Use Kubernetes Service Account to assume IAM role](/solutions/security/cloud/get-started-with-kspm.md#kspm-use-irsa)
* [Use default instance role](/solutions/security/cloud/get-started-with-kspm.md#kspm-use-instance-role)
* [Use access keys directly](/solutions/security/cloud/get-started-with-kspm.md#kspm-use-keys-directly)
* [Use temporary security credentials](/solutions/security/cloud/get-started-with-kspm.md#kspm-use-temp-credentials)
* [Use a shared credentials file](/solutions/security/cloud/get-started-with-kspm.md#kspm-use-a-shared-credentials-file)
* [Use an IAM role ARN](/solutions/security/cloud/get-started-with-kspm.md#kspm-use-iam-arn)

Regardless of which option you use, you’ll need to grant the following permissions:

```console
ecr:GetRegistryPolicy,
eks:ListTagsForResource
elasticloadbalancing:DescribeTags
ecr-public:DescribeRegistries
ecr:DescribeRegistry
elasticloadbalancing:DescribeLoadBalancerPolicyTypes
ecr:ListImages
ecr-public:GetRepositoryPolicy
elasticloadbalancing:DescribeLoadBalancerAttributes
elasticloadbalancing:DescribeLoadBalancers
ecr-public:DescribeRepositories
eks:DescribeNodegroup
ecr:DescribeImages
elasticloadbalancing:DescribeLoadBalancerPolicies
ecr:DescribeRepositories
eks:DescribeCluster
eks:ListClusters
elasticloadbalancing:DescribeInstanceHealth
ecr:GetRepositoryPolicy
```

If you are using the AWS visual editor to create and modify your IAM Policies, you can copy and paste this IAM policy JSON object:

::::{dropdown} Click to view JSON object
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "ecr:GetRegistryPolicy",
                "eks:ListTagsForResource",
                "elasticloadbalancing:DescribeTags",
                "ecr-public:DescribeRegistries",
                "ecr:DescribeRegistry",
                "elasticloadbalancing:DescribeLoadBalancerPolicyTypes",
                "ecr:ListImages",
                "ecr-public:GetRepositoryPolicy",
                "elasticloadbalancing:DescribeLoadBalancerAttributes",
                "elasticloadbalancing:DescribeLoadBalancers",
                "ecr-public:DescribeRepositories",
                "eks:DescribeNodegroup",
                "ecr:DescribeImages",
                "elasticloadbalancing:DescribeLoadBalancerPolicies",
                "ecr:DescribeRepositories",
                "eks:DescribeCluster",
                "eks:ListClusters",
                "elasticloadbalancing:DescribeInstanceHealth",
                "ecr:GetRepositoryPolicy"
            ],
            "Resource": "*"
        }
    ]
}
```

::::



#### Option 1 - [Recommended] Use Kubernetes Service Account to assume IAM role [kspm-use-irsa]

Follow AWS’s [EKS Best Practices](https://aws.github.io/aws-eks-best-practices/security/docs/iam/#iam-roles-for-service-accounts-irsa) documentation to use the [IAM Role to Kubernetes Service-Account](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) (IRSA) feature to get temporary credentials and scoped permissions.

::::{important}
During setup, do not fill in any option in the "Setup Access" section. Click **Save and continue**.
::::



#### Option 2 - Use default instance role [kspm-use-instance-role]

Follow AWS’s [IAM roles for Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) documentation to create an IAM role using the IAM console, which automatically generates an instance profile.

::::{important}
During setup, do not fill in any option in the "Setup Access" section. Click **Save and continue**.
::::



#### Option 3 - Use access keys directly [kspm-use-keys-directly]

Access keys are long-term credentials for an IAM user or AWS account root user. To use access keys as credentials, you must provide the `Access key ID` and the `Secret Access Key`.

For more details, refer to AWS' [Access Keys and Secret Access Keys](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html) documentation.

::::{important}
You must select "Programmatic access" when creating the IAM user.
::::



#### Option 4 - Use temporary security credentials [kspm-use-temp-credentials]

You can configure temporary security credentials in AWS to last for a specified duration. They consist of an access key ID, a secret access key, and a security token, which is typically found using `GetSessionToken`.

Because temporary security credentials are short term, once they expire, you will need to generate new ones and manually update the integration’s configuration to continue collecting cloud posture data. Update the credentials before they expire to avoid data loss.

::::{note}
IAM users with multi-factor authentication (MFA) enabled need to submit an MFA code when calling `GetSessionToken`. For more details, refer to AWS' [Temporary Security Credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html) documentation.
::::


You can use the AWS CLI to generate temporary credentials. For example, you could use the following command if you have MFA enabled:

```console
`sts get-session-token --serial-number arn:aws:iam::1234:mfa/your-email@example.com --duration-seconds 129600 --token-code 123456`
```

The output from this command includes the following fields, which you should provide when configuring the KSPM integration:

* `Access key ID`: The first part of the access key.
* `Secret Access Key`: The second part of the access key.
* `Session Token`: A token required when using temporary security credentials.


#### Option 5 - Use a shared credentials file [kspm-use-a-shared-credentials-file]

If you use different AWS credentials for different tools or applications, you can use profiles to define multiple access keys in the same configuration file. For more details, refer to AWS' [Shared Credentials Files](https://docs.aws.amazon.com/sdkref/latest/guide/file-format.html) documentation.

Instead of providing the `Access key ID` and `Secret Access Key` to the integration, provide the information required to locate the access keys within the shared credentials file:

* `Credential Profile Name`: The profile name in the shared credentials file.
* `Shared Credential File`: The directory of the shared credentials file.

If you don’t provide values for all configuration fields, the integration will use these defaults:

* If `Access key ID`, `Secret Access Key`, and `ARN Role` are not provided, then the integration will check for `Credential Profile Name`.
* If there is no `Credential Profile Name`, the default profile will be used.
* If `Shared Credential File` is empty, the default directory will be used.
* For Linux or Unix, the shared credentials file is located at `~/.aws/credentials`.


#### Option 6 - Use an IAM role Amazon Resource Name (ARN) [kspm-use-iam-arn]

An IAM role Amazon Resource Name (ARN) is an IAM identity that you can create in your AWS account. You define the role’s permissions. Roles do not have standard long-term credentials such as passwords or access keys. Instead, when you assume a role, it provides temporary security credentials for your session. An IAM role’s ARN can be used to specify which AWS IAM role to use to generate temporary credentials.

For more details, refer to AWS' [AssumeRole API](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html) documentation. Follow AWS' instructions to [create an IAM user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html), and define the IAM role’s permissions using the JSON permissions policy above.

To use an IAM role’s ARN, you need to provide either a [credential profile](/solutions/security/cloud/get-started-with-kspm.md#kspm-use-a-shared-credentials-file) or [access keys](/solutions/security/cloud/get-started-with-kspm.md#kspm-use-keys-directly) along with the `ARN role`. The `ARN Role` value specifies which AWS IAM role to use for generating temporary credentials.

::::{note}
If `ARN Role` is present, the integration will check if `Access key ID` and `Secret Access Key` are present. If not, the package will check for a `Credential Profile Name`. If a `Credential Profile Name` is not present, the default credential profile will be used.
::::



### Finish configuring the KSPM integration for EKS [kspm-setup-eks-finish]

Once you’ve provided AWS credentials, finish configuring the KSPM integration:

1. If you want to monitor Kubernetes clusters that aren’t yet enrolled in {{fleet}}, select **New Hosts** under “where to add this integration”.
2. Name the {{agent}} policy. Use a name that matches the purpose or team of the cluster(s) you want to monitor. For example, `IT-dev-k8s-clusters`.
3. Click **Save and continue**, then **Add agent to your hosts**. The **Add agent** wizard appears and provides a DaemonSet manifest `.yaml` file with pre-populated configuration information, such as the `Fleet ID` and `Fleet URL`.


### Deploy the KSPM integration to EKS clusters [kspm-setup-eks-modify-deploy]

The **Add agent** wizard helps you deploy the KSPM integration on the Kubernetes clusters you wish to monitor. For each cluster:

1. Download the manifest and make any necessary revisions to its configuration to suit the needs of your environment.
2. Apply the manifest using the `kubectl apply -f` command. For example: `kubectl apply -f elastic-agent-managed-kubernetes.yaml`

After a few minutes, a message confirming the {{agent}} enrollment appears, followed by a message confirming that data is incoming. You can then click **View assets** to see where the newly-collected configuration information appears throughout {{kib}}, including the [Findings page](/solutions/security/cloud/findings-page-2.md) and the [Cloud Security Posture dashboard](/solutions/security/dashboards/cloud-security-posture-dashboard.md).


## Set up KSPM for unmanaged Kubernetes clusters [kspm-setup-unmanaged]

Follow these steps to deploy the KSPM integration to unmanaged clusters. Keep in mind credentials are NOT required for unmanaged deployments.


### Configure the KSPM integration [_configure_the_kspm_integration]

To install the integration on unmanaged clusters:

1. Find **Cloud Security Posture** in the navigation menu or use the [global search field](/explore-analyze/find-and-organize/find-apps-and-objects.md).
2. Click **Add a KSPM integration**.
3. Read the integration’s description to understand how it works. Then, click [**Add Kubernetes Security Posture Management**](https://docs.elastic.co/en/integrations/cloud_security_posture).
4. Name your integration. Use a name that matches the purpose or team of the cluster(s) you want to monitor, for example, `IT-dev-k8s-clusters`.
5. Select **Unmanaged Kubernetes** from the **Kubernetes Deployment** menu.
6. If you want to monitor Kubernetes clusters that aren’t yet enrolled in {{fleet}}, select **New Hosts** when choosing the {{agent}} policy.
7. Select the {{agent}} policy where you want to add the integration.
8. Click **Save and continue**, then **Add agent to your hosts**. The **Add agent** wizard appears and provides a DaemonSet manifest `.yaml` file with pre-populated configuration information, such as the `Fleet ID` and `Fleet URL`.

:::{image} /solutions/images/security-kspm-add-agent-wizard.png
:alt: The KSPM integration's Add agent wizard
:screenshot:
:::


### Deploy the KSPM integration to unmanaged clusters [kspm-setup-unmanaged-modify-deploy]

The **Add agent** wizard helps you deploy the KSPM integration on the Kubernetes clusters you wish to monitor. To do this, for each cluster:

1. Download the manifest and make any necessary revisions to its configuration to suit the needs of your environment.
2. Apply the manifest using the `kubectl apply -f` command. For example: `kubectl apply -f elastic-agent-managed-kubernetes.yaml`

After a few minutes, a message confirming the {{agent}} enrollment appears, followed by a message confirming that data is incoming. You can then click **View assets** to see where the newly-collected configuration information appears throughout {{kib}}, including the [Findings page](/solutions/security/cloud/findings-page-2.md) and the [Cloud Security Posture dashboard](/solutions/security/dashboards/cloud-security-posture-dashboard.md).


### Set up KSPM on ECK deployments [kspm-eck]

To run KSPM on an [ECK](/deploy-manage/deploy/cloud-on-k8s/deploy-an-orchestrator.md) deployment, you must edit the [Elastic Agent CRD](/deploy-manage/deploy/cloud-on-k8s/configuration-standalone.md) and [Elastic Agent Cluster-Role](/deploy-manage/deploy/cloud-on-k8s/configuration-standalone.md#k8s-elastic-agent-role-based-access-control) `.yaml` files.

::::{dropdown} Patch Elastic Agent
Add `volumes` and `volumeMounts` to `podTemplate`:

```yaml
podTemplate:
  spec:
    containers:
    - name: agent
      volumeMounts:
      - name: proc
        mountPath: /hostfs/proc
        readOnly: true
      - name: cgroup
        mountPath: /hostfs/sys/fs/cgroup
        readOnly: true
      - name: varlibdockercontainers
        mountPath: /var/lib/docker/containers
        readOnly: true
      - name: varlog
        mountPath: /var/log
        readOnly: true
      - name: etc-full
        mountPath: /hostfs/etc
        readOnly: true
      - name: var-lib
        mountPath: /hostfs/var/lib
        readOnly: true
      - name: etc-mid
        mountPath: /etc/machine-id
        readOnly: true
    volumes:
    - name: proc
      hostPath:
        path: /proc
    - name: cgroup
      hostPath:
        path: /sys/fs/cgroup
    - name: varlibdockercontainers
      hostPath:
        path: /var/lib/docker/containers
    - name: varlog
      hostPath:
        path: /var/log
    - name: etc-full
      hostPath:
        path: /etc
    - name: var-lib
      hostPath:
        path: /var/lib
    # Mount /etc/machine-id from the host to determine host ID
    # Needed for Elastic Security integration
    - name: etc-mid
      hostPath:
        path: /etc/machine-id
        type: File
```

::::


::::{dropdown} Patch RBAC
Make sure that the `elastic-agent` service-account has the following Role and ClusterRole:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: default
  name: elastic-agent
subjects:
- kind: ServiceAccount
  name: elastic-agent
  namespace: default
roleRef:
  kind: Role
  name: elastic-agent
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: elastic-agent
  labels:
    k8s-app: elastic-agent
rules:
- apiGroups: [""]
  resources:
  - nodes
  - namespaces
  - events
  - pods
  - services
  - configmaps
  - serviceaccounts
  - persistentvolumes
  - persistentvolumeclaims
  verbs: ["get", "list", "watch"]
- apiGroups: ["extensions"]
  resources:
  - replicasets
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources:
  - statefulsets
  - deployments
  - replicasets
  - daemonsets
  verbs: ["get", "list", "watch"]
- apiGroups:
  - ""
  resources:
  - nodes/stats
  verbs:
  - get
- apiGroups: [ "batch" ]
  resources:
  - jobs
  - cronjobs
  verbs: [ "get", "list", "watch" ]
- nonResourceURLs:
  - "/metrics"
  verbs:
  - get
- apiGroups: ["rbac.authorization.k8s.io"]
  resources:
  - clusterrolebindings
  - clusterroles
  - rolebindings
  - roles
  verbs: ["get", "list", "watch"]
- apiGroups: ["policy"]
  resources:
  - podsecuritypolicies
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: elastic-agent
  namespace: default
  labels:
    k8s-app: elastic-agent
rules:
  - apiGroups:
    - coordination.k8s.io
    resources:
    - leases
    verbs: ["get", "create", "update"]
```

::::
