# K8s Training Clusters Everywhere via Terraform

This is a collection of scripts created to provision K8s clusters on a collection of cloud providers as quick three-node test clusters to practice for the [CKA][cka] and [CKAD][ckad] exams.  (Well, that, and to learn more about [Terraform][tf])  Creating clusters is easy, so the real value of the code here is quickly deleting resources.  If, like me, you've used up your free cloud credits and don't want to play games to get more, it's good to have a reliable way to clean up and insure you're not running things unnecessarily.

**Note:** I'm strictly using Terraform's [latest JSON syntax][tf_json] for all plans, so code here is expected to be used with TF 0.12+.  It may work with earlier versions, or it may not.

## Usage

First, `tf init` in the root of this repo to get all the providers downloaded and set up.  As updates are made later, it may be important to `init` again to keep things working.  Fortunately, Terraform will let you know if you need to do so.

Next, if you just `terraform apply` and approve, then -- assuming you've supplied the necessary var file(s) -- Terraform will simply provision a base cluster on every included platform (peek in `modules` for those), which is probably not what you want.  Instead, to get just one target cluster created, specify the module corresponding to the cloud provider to which to want to deploy.  For example, to run just the Digital Ocean module, execute `terraform apply -target=module.do`.

Is this the right way to go about this?  Probably not, but I have other priorities above breaking things into multiple repos or a creating a repo full of separate TF plans right now, so I figure it'll do until I get around to changing things.  That said, pull request are always welcome.  😁

**Another note:** Change of plans -- Since Azure has K8s 1.17, I've stashed the work on GCP for now, and focusing on Azure first.

### Variables

As hinted at above, proper variable file(s) will have to be supplied for things to run at all.  The easiest thing to do is to create a `.tfvars` file with all necessary variables the root directory of this repo.  The variables used for each module are as follows:

#### Digital Ocean (DO)

- `do_token` -- An authentication token used to authenticate to the DO API.  These are generated on the **Applications & API** page accessed via **API** link on the left nav menu on your [DO console][do_console].
- `do_region` -- The "slug" indicating the datacenter region in which to deploy the cluster.  Available regions can be found via [DO's nifty CLI tool][doctl] (`doctl k o r`), or on the [DO site][do_reg_slugs].
- `do_k8s_version` -- The "slug" indicating the version of Kubernetes to deploy.  Like regions, available versions can also be found via the [CLI tool][doctl] (`doctl k o v`), or on the [DO site][do_k8s_slugs].

#### Google Cloud Platform (GCP)

TBD

#### Microsoft Azure

_Refer to the Azure [README.md](modules/azure/README.md) for more on these first two:_

- `az_sp_id` -- An Azure service prinicipal ID (GUID).
- `az_sp_secret` -- The secret for the above Azure service principal.
- `az_region` -- An Azure region where the cluster will be provisioned.  The list of regions is available on the [Azure docs site][az_regions].
- `az_rg` -- An Azure resource group where all the cluster resources will be provisioned.  Will be created if it doesn't already exist, and destroyed with everything else in a `terraform deploy`, so **be careful about using an existing RG with other resources in it!**
- `az_prefix` -- Used to name the cluster and as DNS prefix for the cluster.
- `az_vm_size` -- Azure VM size to use for all nodes.
- `az_k8s_version` -- The version of Kubernetes to deploy.  The best place to find valid values for a given region is via the Azure CLI: `az aks get-versions --location eastus --output table`

_This last one is just a tag I use on Azure, and isn't actually required by the Terraform resource -- just by me._

- `az_environment` -- (Ideally) One of Production, QA, Development, etc.  

#### Amazon Web Services (AWS)

TBD

## K8s Config

The plan right now is to have each module create a `kubeconfig.yaml` file in it's directory when it's plan is applied, and delete the file when the resources are destroyed.  That way, it's just a matter of `export KUBECONFIG=path/to/module/kubeconfig.yaml`-ing to get going.

**Yet another note:** It was easy to find the kubeconfig content in the DO provider API, so I'm assuming I'll be able to do the same for the other providers as well.  More to come...

[cka]: https://www.cncf.io/certification/cka/
[ckad]: https://www.cncf.io/certification/ckad/
[tf]: https://www.terraform.io/
[tf_json]: https://www.terraform.io/docs/configuration/syntax-json.html
[tf_mods]: https://www.terraform.io/docs/modules/index.html
[do_console]: https://cloud.digitalocean.com/projects
[doctl]: https://github.com/digitalocean/doctl
[do_reg_slugs]: https://www.digitalocean.com/docs/platform/availability-matrix/
[do_k8s_slugs]: https://slugs.do-api.dev/
[az_regions]: https://azure.microsoft.com/en-us/global-infrastructure/services/?products=kubernetes-service
[az_]: 
