# kodekloud-Sealed-secret


Sealed secrets is , it is a tool that you can use to help manage your secrets and it can work across a variety of different enviroments and plateforms like kubernetes or terraform, 

Why do we need Sealed Secrets
-----------------------------

so let's say that we are working witha a kubernetes environment and we need our application to connect to a database. and so we need to provide the password to connect to the database. And usually we are gonna do that with a secret, And so in kubernetes, We have 2 ways of creating secrets.

**imperative and declarative way**

Problem
--------------

#secret.example.in.kubernetes
    
    apiVersion: v1
    data:
        DB_PASSWORD: cGFzc3dvcmQzMjM=        --> password of database in base64 encoded
    kind: Secret
    metadata:
        name: database
        namespace:  default

The problem with this is that you could see and example of a secret being created. you could see we have got the DB_PASSWORD which represent the password. We would connect to our database. **Now, you create the secret, what is gonna happen is the password is going to be base64 encoded(remember secrets are not encrypted, it is base64 encoded.) which mean that it is not secret and anybody else can decode it if they get access to your kubernetes manifest**

matlab y ha k kubernetes ma jo ap secret create krty hn wo base64 encoded hoty hn jo k decode ho sakhty hn... so y secure ni hoty, jis kisi ki b access apki kubernetes manifest per ha wo apka database or koi ma secreted secrets k password decode per k dekh sakhta tha... or agr ap apny templete ko github per rakhty hn tu koi b apky secrets dekha sakhta ha.. so y secure way ni ha...


Sealed Secrets and its Components
--------------------------------

**So, what sealed secrets does is, it is ultimately allows you to encrypt secrets that you can safely store in a public repository.**

    so we can actually encrypt our secrets stored in our manifest and upload it to github and not have to worry about anybody getting access to our secrets because it is going to be encrypted while, it is in our Github repo. Now it is sealed secrets,

There is 3 main components that we work with. 

-   Operator/controller    there is a sealed secrets operator that you are gonna deploy onto your kubernetes cluster and that going to be responsible for actually decrypting your secrets.

-   Q Cube Seal      you have got CLI tool called **Q Cube Seal** and so what the purpose of this CLI tool is it takes your secret, **and it is the one that does the encrypting**, so the CLI tool encrypts it, and then once the operator/controller sees the encrypted secret, it will then perform the decryption and then pass it off to your containers.

-   Custom Resource Definition(CRD)     so instead of creating regular secrets, you create sealed secrets, which is a CRD within kubernetes, and that how the operator knows when to perform the decryption process within your kubernetes cluster.

How does Sealed Secrets work
----------------------------

So how sealed secrets works to deploy sealed secrets..

**The first thing that we have to do is we actually have to deploy the sealed operator onto kubernetes cluster. Now, you can deploy the manifest by itself, or you can make use of Helm to actually go ahead and deploy the sealed secrets operator for you.**

Now when you deploy the sealed secrets operator in kubernetes cluster, there is going to be a sealed secret CRD. 

So this is the new object that we are gonna be working with. so we will no longer upload manifests with a regular secret.

we will always upload a sealed secret, which is just a secret that encrypted. And on top of that, the operator's also going to create a public key and a private key. So this, 

these are the keys that are gonna be used for encrypting and decrypting. Alright, so at this point, the operator has been deployed and everything is good to go.

so now it is a matter of how exactly do we encrypt our secrets? 

so let say we have a secret.

**what we are gonna do is we are gonna utilixe the cube seal CLI, and we are gonna pass in a regular kubernetes secret.**

**and what this is going to do is by utilizing the public key that was generated by the operator, we can then encrypted it and create a sealed secret object.**

remember, this is the CRD that was created by the operator. so this is a brand new kubernetes object, and it is going to look something like this, right

it is just like any other kubernetes object, but it representes a encrypted secret.

And what we are gonna do is we can then go ahead and apply this like any other object.

So we will do a cube CTL apply.

**and then that going to create the sealed object in our kubernetes cluster. and at this point, the operator will detect a new sealed secret getting created.**

**and what it is going do is it is going to take the private key that it created, and it is going to decrypt that secret**

and it is foing to return a regular secret on our kubertes cluster, so when you actually log into your k8, you could do a cude, 

kubectl get secret.. 

and you will see a regular secret. at that point. you can then pass it as a env to your container or mounted as a volume. it works just like any other secret that you would normally work with. but the benefit of this solution is that when you go to upload your manifest.. to a github repo. it is going to have a sealed secrets, not the regular secrets. And remember, the sealed secrets have already been encrypted..

**and the only way you can decrypt it is with the private key. which is safely stored within our kubernetes cluster**

So you could feel safe and confident that nobody will have access to your secrets even if they have access to your GITHUB. 

Flow:
----

    normal-secret --> Q cube CLI(kubesealed)  --> it takes public key from operator/controller ---> then uses this key and convert normal secret to sealed secret.(make it encrypt)

# sealed secret template

    apiVersion: bitnami.com/v1alpha1
    kind:   SealedSecret
    metadata:
        name:   database
        namespace:  mynamespace
    spec:
        encryptedData:
            DB_PASSWORD:
            <SEALED-SECRET-IN-ENCRYPTED-FORM>
    
    Then APPLY:
    
        kubectl apply ---   > and it will create sealed secret object in kubernetes cluster.. then inside the cluster operator will see a new sealed secret and it will decrypt it by using private key(remember operator hi dono keys create krta ha public deta ha Q CUBE CLI ko sealed secret bny k lye, or jb sealed secret bn kr cluster ma as object ata ha tu isko apni private key sa decrypt kerta ha..) or wo phir isko kubernetes cluster k regular secrets ma rakh dye ga...
    
        so jb ap cluster ko login kry gye tu ap 
        
        kubectl get secret ko use kr k secret dekh sakhty hn. phir ap isko as env container ko pass kr sakhty hn. ab y as normal secret jis tarha work kerta ha wasy hi work kry ga...
    

benefit:
-------

jb ap apni manifest ko github repo per upload kry gye tu apka secret encrypted form ma uplaod hoga.(sealed secrets)

DEMO WORKING WITH SEALED SECRETS
---------------------------------

    1- First thing:
    
        deploy the sealed secret operator.. in cluster.. ap y kam **kubectl apply mean manifest" or "helm chart** k through ker sakhty hn. 
    
        make sure you have helm installed...
    
        $ helm repo add sealed-secrets https://bitnami-labs.githubs.io/sealed-secrets
    
        $ helm install my-release sealed-secrets/sealed-secrets  ---> it will install in default namespace, other namespace k lye command ma namespace mention kro...
    
    
    **kube-system namespace ma ap sealed secret operator ka pod running ma dekh sakhty hn..** mean it is deploy successfully...
    
Installation of kube sealed CLI
-------------------------------

you can see it instruction from github page of sealed secret.

A client-side utility: kubeseal

FOR Linux:
----------

    The kubeseal client can be installed on Linux, using the below commands:

    KUBESEAL_VERSION='' # Set this to, for example, KUBESEAL_VERSION='0.23.0'
    curl -OL "https://github.com/bitnami-labs/sealed-secrets/releases/download/v${KUBESEAL_VERSION:?}/kubeseal-${KUBESEAL_VERSION:?}-linux-amd64.tar.gz"
    tar -xvzf kubeseal-${KUBESEAL_VERSION:?}-linux-amd64.tar.gz kubeseal

    sudo install -m 755 kubeseal /usr/local/bin/kubeseal
    
If you have curl and jq installed on your machine, you can get the version dynamically this way. This can be useful for environments used in automation and such.

    # Fetch the latest sealed-secrets version using GitHub API
    KUBESEAL_VERSION=$(curl -s https://api.github.com/repos/bitnami-labs/sealed-secrets/tags | jq -r '.[0].name' | cut -c 2-)

    # Check if the version was fetched successfully
    if [ -z "$KUBESEAL_VERSION" ]; then
        echo "Failed to fetch the latest KUBESEAL_VERSION"
        exit 1
    fi

    curl -OL "https://github.com/bitnami-labs/sealed-secrets/releases/download/v${KUBESEAL_VERSION}/kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz"
    tar -xvzf kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz kubeseal
    sudo install -m 755 kubeseal /usr/local/bin/kubeseal
    
where KUBESEAL_VERSION is the version tag of the kubeseal release you want to use. For example: v0.18.0.

Installation from source
------------------------

    If you just want the latest client tool, it can be installed into $GOPATH/bin with:

    go install github.com/bitnami-labs/sealed-secrets/cmd/kubeseal@main

You can specify a release tag or a commit SHA instead of main.

The go install command will place the kubeseal binary at $GOPATH/bin:

    $(go env GOPATH)/bin/kubeseal


Establishing connection b/w kubeseal and kubernetes cluster
-----------------------------------------------------------

Kubeseal need to be able to access public key created by our sealed secret operator..

so way kya ha test kerny ka k public key kubeseal retrive kr rha ha operator sa k ni...

command:

    first get the operator/controller service name that you have already been installed within your cluster. Because you need to use it on further command. other wise it would not work..

    like:

    kubectl get svc -n kube-system    --> and get sealed service name from here..

now

    kubeseal --fetch-cert --controller-name <sealed service name get from kube-system namespace> --controller-namespace kube-system

isky return ma apko certificate mily ga.. jo k proof ha k apka installed kya hoga **kubeseal** linux system ma, k8 cluster sa installed operator/controller sa communicated ker pa rha ha... or us sa public key lye pye ga... 

or public key lye pye ga tu mean wo phir usko use ker k sealed secret ma bna ker dye sakhta ha..

Creating Kubernetes Secret
--------------------------

Creating Kubernetes Secret and pass it to kubeseal for creating sealed secrets..

so create a kubernetes secret template file...

    kubectl create secret generic database -n default --from-literal=DB_PASSWORD=password123 --dry-run=client -o yaml > secret.yaml

    it will give a template like this..


    apiVersion: v1
    kind:   Secret
    metadata:   
        creationTimestamp: null
        name: database
        namespace: default
    data:
        DB_PASSWORD: <password in base64 format>

apka secret template ma encoded form ma hoga jo k easy decode ma sakhtya ha... mean y secure ni ha...

so make it secure by giving it to kubesealed(Encrypting Secret)
-----------------------------

command:

    kubeseal --controller-name <sealed service name get from kube-system namespace> --controller-namespace kube-system --format yaml <"secret.yaml"(name of the k8 secret file template) > sealed-secret.yaml


       kubeseal --controller-name <sealed service name get from kube-system namespace> --controller-namespace kube-system --format yaml < secret.yaml > sealed-secret.yaml

jb ap isko run kry gye tu output file ma apko y dye ga...

    apiVersion: bitnami.com/v1alpha1
    kind:   SealedSecret
    metadata:
        creationTimestamp: null
        name: database
        namespace:  default
    spec:
        encryptedData:
            DB_PASSWORD: <>

ab apky secret template k kind secret ni bulky sealedsecret hoga...jb ap is template ko kubectl apply kry gye tu cluster ma isky name k object create hojye ga...

ap isko github repo ma b ab rakh sakhty hn because secret ab encrypted form ma ha..

Applying
-------------

    kubectl apply -f <sealsecretfilename>

    kubectl apply -f sealed-secret.yaml

ap apply k bd cluster ma iska object bna or operator na dekha k new object sealed secret k bna ha, so is na apply private ko use ker k isko decrypt ker dya(y kam background ma horha hota ha automatically)... or ab y ap k cluster ma normal secret object ki tarha hoga...

jis ko ap kubectl get secret ker k b dekh sakhty hn

like:

    kubectl get secret <secret name> -o yaml

    or ap isky password ko kubernetes cluster k under sa lye ker **decode**
    b ker sakhty hn..

    like:

        echo <password get from kubernetes cluster secret> | base64 --decode


mean cluster ma bahir outer world ma y ap save ha ap isky template ko github or kisi b public plateform per save ker sakhty hn... phily kerty tu wo easily decode hojata or password expose hojata..

ab ap na kubeseal k through password encrypt ker dya ha tu ab public platefrom per rakho gye b tu y decrypt ni hoga,.. because decrpyt kerny k lye private key sirf apky cluster ma operator k pass. so ap isko kisi b public plateform per rakha sakhty hn...

cluster ma ja kr seal secret as normal secret behave kerta ha.. so ap waha sa encode secret ko decode ker skahty hn...  

command to see sealed object
----------------------------

    kubectl get sealedsecret

    kubectl describe sealedesecret <sealedsecret name>
