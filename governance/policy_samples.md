# Red Hat Advanced Cluster Management for Kubernetes configuration policy samples

View a description of the configuration policy samples that are included in your Red Hat Advanced Cluster Management for Kubernetes installation.

* [Memory usage policy](#memory-usage-policy) 
* [Namespace policy](#namespace-policy)
* [Image vulnerability policy](#image-vulnerability-policy)
* [Pod nginx policy](#pod-nginx-policy)
* [Pod security policy](#pod-security-policy)
* [Role policy](#role-policy)
* [Rolebinding policy](#rolebinding-policy)
* [Security context constraints (SCC) policy](#security-context-constraints-scc-policy) 


**Note**: You must create a placement policy and placement binding policy to apply your policies on to your cluster. For more information, see [Red Hat Advanced Cluster Management for Kubernetes policy example](../governance/policy_example.md).

## Memory usage policy

Apply the limit range policy to limit or restrict your memory and compute usage. For more information, see _Limit Ranges_ in the [Kubernetes documentation](https://kubernetes.io/docs/concepts/policy/limit-range/). Your memory usage policy might resemble the following YAML file:

   ```yaml
   apiVersion: policy.mcm.ibm.com/v1alpha1
   kind: Policy
   metadata:
     name: policy-limitrange
     namespace: mcm 
   spec:
     complianceType: musthave
     remediationAction: inform
     namespaces:
       exclude: ["kube-*"]
       include: ["default"]
     object-templates:
       - complianceType: musthave
         objectDefinition:
           apiVersion: v1
           kind: LimitRange # limit memory usage
           metadata:
             name: mem-limit-range
           spec:
             limits:
             - default:
                 memory: 512Mi
               defaultRequest:
                 memory: 256Mi
               type: Container
           ...
   ```
   
### Applying the memory usage policy

Complete the following steps to apply the memory usage policy from the console:

1. Log in to your Red Hat Advanced Cluster Management for Kubernetes console.
2. From the navigation menu, click **Governance and risk**. 
3. Click **Create policy**. 
4. Select **Limitrange** from the _Specifications_ field.

## Namespace policy

Apply the namespace policy to define specific rules for your namespace. Your namespace policy might resemble the following YAML file:
  
   ```yaml
   apiVersion: policy.mcm.ibm.com/v1alpha1
   kind: Policy
   metadata:
     name: policy-namespace-1
     namespace: mcm 
   spec:
     complianceType: musthave
     remediationAction: inform
     namespaces:
       exclude: ["kube-*"]
       include: ["default"]
     object-templates:
       - complianceType: musthave
         objectDefinition:
           kind: Namespace # must have namespace 'prod'
           apiVersion: v1
           metadata:
             name: prod
        ...
   ```
   
### Applying the namespace policy

Complete the following steps to apply the memory usage policy from the console:

1. Log in to your Red Hat Advanced Cluster Management for Kubernetes console.
2. From the navigation menu, click **Governance and risk**. 
3. Click **Create policy**. 
4. Select **Namespace** from the _Specifications_ field.

        
## Image vulnerability policy

Apply the image vulnerability policy to detect if container images have vulnerabilities by leveraging the Container Security Operator. For more information about the Security Operator, see the _Container Security Operator_ from the [Quay repository](https://github.com/quay/container-security-operator). The policy installs the Container Security Operator on your managed cluster if it is not installed. 

Your image vulnerability policy might resemble the following YAML file:

   ```yaml
   apiVersion: policy.mcm.ibm.com/v1alpha1
   kind: Policy
   metadata:
     name: policy-imagemanifestvulnpolicy
     namespace: default
     annotations:
       policy.mcm.ibm.com/standards: NIST-CSF
       policy.mcm.ibm.com/categories: DE.CM Security Continuous Monitoring
       policy.mcm.ibm.com/controls: DE.CM-8 Vulnerability scans
   spec:
     complianceType: musthave
     remediationAction: inform
     disabled: false
     namespaces:
       exclude: ["kube-*"]
       include: ["*"]
     object-templates:
       - complianceType: musthave
         objectDefinition:
           apiVersion: operators.coreos.com/v1alpha1
           kind: ClusterServiceVersion
           metadata:
             annotations:
               capabilities: Full Lifecycle
               categories: Security
               containerImage: quay.io/quay/container-security-operator@sha256:15a4b50d847512b5f404ec1cf72c30c98e073a7f26f1588213bd2e8b6331f016
               createdAt: 2019-11-16 01:03:00
               description: Identify image vulnerabilities in Kubernetes pods
               repository: https://github.com/quay/container-security-operator
               tectonic-visibility: ocs
             name: container-security-operator.v1.0.1
             namespace: openshift-operators
           spec:
             customresourcedefinitions:
               owned:
               - description: Represents a set of vulnerabilities in an image manifest.
                 displayName: Image Manifest Vulnerability
                 kind: ImageManifestVuln
                 name: imagemanifestvulns.secscan.quay.redhat.com
                 version: v1alpha1
             description: "The Container Security Operator (CSO) brings Quay and Clair metadata to Kubernetes / OpenShift.\
               \ Starting with vulnerability information the scope will get expanded over time. If it runs on OpenShift,\
               \ the corresponding vulnerability information is shown inside the OCP Console. The Container Security Operator\
               \ enables cluster administrators to monitor known container\
               \ image vulnerabilities in pods running on their Kubernetes cluster. The controller sets up a watch\
               \ on pods in the specified namespace(s) and queries the container registry for vulnerability\
               \ information. If the container registry supports image scanning,\
               \ such as [Quay](https://github.com/quay/quay) with [Clair](https://github.com/quay/clair),\
               \ then the Operator will expose any vulnerabilities found via the Kubernetes API in an\
               \ `ImageManifestVuln` object.  This Operator requires no additional configuration after deployment,\
               \ and will begin watching pods and populating `ImageManifestVulns` immediately once installed."
             displayName: Container Security
             install:
               spec:
                 deployments:
                 - name: container-security-operator
                   spec:
                     replicas: 1
                     selector:
                       matchLabels:
                         name: container-security-operator-alm-owned
                     template:
                       metadata:
                         labels:
                           name: container-security-operator-alm-owned
                         name: container-security-operator-alm-owned
                       spec:
                         containers:
                         - command:
                           - /bin/security-labeller
                           - '--namespaces=$(WATCH_NAMESPACE)'
                           env:
                           - name: MY_POD_NAMESPACE
                             valueFrom:
                               fieldRef:
                                 fieldPath: metadata.namespace
                           - name: MY_POD_NAME
                             valueFrom:
                               fieldRef:
                                 fieldPath: metadata.name
                           - name: WATCH_NAMESPACE
                             valueFrom:
                               fieldRef:
                                 fieldPath: "metadata.annotations['olm.targetNamespaces']"
                           image: quay.io/quay/container-security-operator@sha256:15a4b50d847512b5f404ec1cf72c30c98e073a7f26f1588213bd2e8b6331f016
                           name: container-security-operator
                         serviceAccountName: container-security-operator
                 permissions:
                 - rules:
                   - apiGroups:
                     - secscan.quay.redhat.com
                     resources:
                     - imagemanifestvulns
                     - imagemanifestvulns/status
                     verbs:
                     - '*'
                   - apiGroups:
                     - ''
                     resources:
                     - pods
                     - events
                     verbs:
                     - '*'
                   - apiGroups:
                     - ''
                     resources:
                     - secrets
                     verbs:
                     - get
                   serviceAccountName: container-security-operator
               strategy: deployment
             installModes:
             - supported: true
               type: OwnNamespace
             - supported: true
               type: SingleNamespace
             - supported: true
               type: MultiNamespace
             - supported: true
               type: AllNamespaces
             keywords:
             - open source
             - containers
             - security
             labels:
               alm-owner-container-security-operator: container-security-operator
               operated-by: container-security-operator
             links:
             - name: Operator Source Code
               url: https://github.com/quay/container-security-operator
             - name: Source Code
               url: https://github.com/quay/container-security-operator
             icon:
             - base64data:    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
               mediatype: image/png
             maturity: alpha
             maintainers:
             - email: quay-devel@redhat.com
               name: Quay Engineering Team
             provider:
               name: Red Hat
             selector:
               matchLabels:
                 alm-owner-container-security-operator: container-security-operator
                 operated-by: container-security-operator
             version: 1.0.1
             replaces: container-security-operator.v1.0.0
       - complianceType: musthave
         objectDefinition:
           apiVersion: operators.coreos.com/v1alpha1
           kind: Subscription
           metadata:
             name: container-security-operator
             namespace: openshift-operators
           spec:
             channel: alpha
             installPlanApproval: Automatic
             name: container-security-operator
             source: community-operators
             sourceNamespace: openshift-marketplace
             startingCSV: container-security-operator.v1.0.1
       - complianceType: mustnothave
         objectDefinition:
           apiVersion: secscan.quay.redhat.com/v1alpha1
           kind: ImageManifestVuln     
   ```
 
### Applying the image vulnerability policy

Complete the following steps to apply the image vulnerability policy from the console:

1. Log in to your Red Hat Advanced Cluster Management for Kubernetes console.
2. From the navigation menu, click **Govern risk**. 
3. Click **Create policy**. 
4. Select **ImageManifestVulnPolicy** from the _Specifications_ field.

## Pod nginx policy

Apply the pod policy to define the container rules for your pods. A nginx pod must exist in your cluster. Your pod policy nginx policy might resemble the following YAML file:

   ```yaml
   apiVersion: policy.mcm.ibm.com/v1alpha1
   kind: Policy
   metadata:
     name: policy-pod
     namespace: mcm 
   spec:
     complianceType: musthave
     remediationAction: inform
     namespaces:
       exclude: ["kube-*"]
       include: ["default"]
     object-templates:
       - complianceType: musthave
         objectDefinition:
           apiVersion: v1
           kind: Pod # nginx pod must exist
           metadata:
             name: nginx-pod
           spec:
             containers:
             - image: nginx:1.7.9
               name: nginx
               ports:
               - containerPort: 80
        ...
   ```

### Applying the pod policy

Complete the following steps to apply the pod policy from the console:

1. Log in to your Red Hat Advanced Cluster Management for Kubernetes console
2. From the navigation menu, click **Govern risk**. 
3. Click **Create policy**. 
4. Select **Pod** from the _Specifications_ field.

## Pod security policy

Apply a pod security policy to secure pods and containers. For more information, see _Pod Security Policies_ in the [Kubernetes documentation](https://kubernetes.io/docs/concepts/policy/pod-security-policy/).

Your pod security policy might resemble the following YAML file:
 
   ```yaml
   apiVersion: policy.mcm.ibm.com/v1alpha1
   kind: Policy
   metadata:
     name: policy-podsecuritypolicy
     namespace: mcm
   spec:
     complianceType: musthave
     remediationAction: inform
     namespaces:
       exclude: ["kube-*"]
       include: ["default"]
     object-templates:
       - complianceType: musthave
         objectDefinition:
           apiVersion: policy/v1beta1
           kind: PodSecurityPolicy # no privileged pods
           metadata:
             name: restricted-mcm
             annotations:
            seccomp.security.alpha.kubernetes.io/allowedProfileNames: '*'
           spec:
             privileged: false # no priviliedged pods
             allowPrivilegeEscalation: false
             allowedCapabilities:
             - '*'
             volumes:
             - '*'
             hostNetwork: true
             hostPorts:
             - min: 1000 # ports < 1000 are reserved
               max: 65535
             hostIPC: false
             hostPID: false
             runAsUser:
               rule: 'RunAsAny'
             seLinux:
               rule: 'RunAsAny'
             supplementalGroups:
               rule: 'RunAsAny'
             fsGroup:
               rule: 'RunAsAny'
        ...
   ```
   
### Applying a pod security policy

Complete the following steps to apply the pod security policy from the console:

1. Log in to your Red Hat Advanced Cluster Management for Kubernetes console
2. From the navigation menu, click **Govern risk**. 
3. Click **Create policy**. 
4. Select **Podsecuritypolicy** from the _Specifications_ field.

## Role policy

Apply a role policy to set rules and permissions for specific roles in your cluster. For more information on roles, see [Role-based access control (RBAC)](../governance/security.md). Your role policy might resemble the following YAML file:

   ```yaml
   apiVersion: policy.mcm.ibm.com/v1alpha1
   kind: Policy
   metadata:
     name: policy-role
     namespace: mcm
   spec:
     complianceType: musthave
     remediationAction: inform
     namespaces:
       exclude: ["kube-*"]
       include: ["default"]
     role-templates:
       - apiVersion: roletemplate.mcm.ibm.com/v1alpha1 # role must follow defined permissions
         metadata:
           namespace: "" # will be inferred
           name: operator-role-policy 
         selector:
           matchLabels:
             dev: "true"
         complianceType: musthave # at this level, it means the role must exist with the rules that it must have the following
         rules:
           - complianceType: musthave # at this level, it means if the role exists the rule is a musthave
             policyRule:
               apiGroups: ["extensions", "apps"]
               resources: ["deployments"]
               verbs: ["get", "list", "watch", "create", "delete","patch"]
          - complianceType: "mustnothave" # at this level, it means if the role exists the rule is a mustnothave
            policyRule:
              apiGroups: ["core"]
              resources: ["secrets"]
              verbs: ["get", "list", "watch","delete", "create", "update", "patch"]
         ...
   ```

### Applying a role policy

Complete the following steps to apply the role policy from the console:

1. Log in to your Red Hat Advanced Cluster Management for Kubernetes console
2. From the navigation menu, click **Govern risk**. 
3. Click **Create policy**. 
4. Select **Role** from the _Specifications_ field.

## Rolebinding policy

Apply a role binding policy to bind a policy to a namespace in your cluster. Your role binding policy might resemble the following YAML file:

   ```yaml
   apiVersion: policy.mcm.ibm.com/v1alpha1
   kind: Policy
   metadata:
     name: policy-rolebinding
     namespace: mcm
   spec:
     complianceType: musthave
     remediationAction: inform
     namespaces:
       exclude: ["kube-*"]
       include: ["default"]
     object-templates:
       - complianceType: musthave
         objectDefinition:
           kind: RoleBinding # role binding must exist
           apiVersion: rbac.authorization.k8s.io/v1
           metadata:
             name: operate-pods-rolebinding
           subjects:
           - kind: User
             name: admin # Name is case sensitive
             apiGroup: rbac.authorization.k8s.io
           roleRef:
             kind: Role #this must be Role or ClusterRole
             name: operator # this must match the name of the Role or ClusterRole you wish to bind to
             apiGroup: rbac.authorization.k8s.io
       ...
   ```
   
### Apply a role policy

Complete the following steps to apply the role binding policy from the console:

1. Log in to your Red Hat Advanced Cluster Management for Kubernetes console
2. From the navigation menu, click **Govern risk**. 
3. Click **Create policy**. 
4. Select **Rolebinding** from the _Specifications_ field.
          

## Security context constraints (SCC) policy

Apply a Security context constraints (SCC) policy to control permissions for pods by defining conditions in the policy. For more information see, [Managing Security Context Constraints (SCC)](https://docs.openshift.com/container-platform/4.3/authentication/managing-security-context-constraints.html#security-context-constraints-about_configuring-internal-oauth).  Your SCC policy might resemble the following YAML file:


   ```
   apiVersion: policy.mcm.ibm.com/v1alpha1
   kind: Policy
   metadata:
     name: policy-scc
     namespace: open-cluster-management-policies
     annotations:
       policy.mcm.ibm.com/standards: NIST-CSF
       policy.mcm.ibm.com/categories: PR.PT Protective Technology
       policy.mcm.ibm.com/controls: PR.PT-3 Least Functionality
   spec:
     complianceType: musthave
     remediationAction: inform
     disabled: false
     namespaces:
       exclude: ["kube-*"]
       include: ["default"]
     object-templates:
       - complianceType: musthave
         objectDefinition:
           apiVersion: security.openshift.io/v1
           kind: SecurityContextConstraints # restricted scc
           metadata:
             annotations:
               kubernetes.io/description: restricted denies access to all host features and requires pods to be run with a UID, and SELinux context that are allocated to the namespace.  This is the most restrictive SCC and it is used by default for authenticated users.
             name: sample-restricted-scc
           allowHostDirVolumePlugin: false
           allowHostIPC: false
           allowHostNetwork: false
           allowHostPID: false
           allowHostPorts: false
           allowPrivilegeEscalation: true
           allowPrivilegedContainer: false
           allowedCapabilities: []
           defaultAddCapabilities: []
           fsGroup:
             type: MustRunAs
           groups:
           - system:authenticated
           priority: null
           readOnlyRootFilesystem: false
           requiredDropCapabilities:
           - KILL
           - MKNOD
           - SETUID
           - SETGID
           runAsUser:
             type: MustRunAsRange
           seLinuxContext:
             type: MustRunAs
           supplementalGroups:
             type: RunAsAny
           users: []
           volumes:
           - configMap
           - downwardAPI
           - emptyDir
           - persistentVolumeClaim
           - projected
           - secret
   ---
   apiVersion: mcm.ibm.com/v1alpha1
   kind: PlacementBinding
   metadata:
     name: binding-policy-scc
     namespace: open-cluster-management-policies
   placementRef:
     name: placement-policy-scc
     kind: PlacementRule
     apiGroup: apps.open-cluster-management.io
   subjects:
   - name: policy-scc
     kind: Policy
     apiGroup: policy.mcm.ibm.com
   ---
   apiVersion: mcm.ibm.com/v1alpha1
   kind: PlacementBinding
   metadata:
     name: policy-scc-production-clusters
     namespace: open-cluster-management-policies
   placementRef:
     name: production-clusters
     kind: PlacementRule
     apiGroup: apps.open-cluster-management.io
   subjects:
   - name: policy-scc
     kind: Policy
     apiGroup: policy.mcm.ibm.com
   ---
   apiVersion: apps.open-cluster-management.io/v1
   kind: PlacementRule
   metadata:
     name: placement-policy-scc
     namespace: open-cluster-management-policies
   spec:
     clusterConditions:
     - type: OK
     clusterSelector:
       matchExpressions: []
  ```

### Appliying the SCC policy

Complete the following steps to apply the SCC policy from the console:

1. Log in to your Red Hat Advanced Cluster Management for Kubernetes console.
2. From the navigation menu, click **Govern risk**. 
3. Click **Create policy**. 
4. Select **SecurityContextConstraintPolicy** from the _Specifications_ field. 


See [Red Hat Advanced Cluster Management for Kubernetes Governance and risk](../governance/compliance_intro.md) for more information about policies.