# Secrets in a workspace

<blockquote class="info">
This article explains how to use secrets in a workspace. To authenticate the
workspace provisioner, see the article on <a href="/admin/auth">Authentication</a>.
</blockquote>

Coder is open-minded about how you get your secrets into your workspaces.
This means you can continue to use the same workflow and tools you already use to manage secrets locally can be brought over to your Coder workspace. 

Usually this workflow looks like,

1. Give your users their secrets in advance
1. Your users write them to a persistent file after they've built their
   workspace
   
<blockquote class="warning">
Its strongly recommended that you do not use <a href="./templates/parameters.md">template parameters</a> to accept
secrets. Parameters are shown in cleartext in Coder. Assume anyone with
view access to a workspace can also see its parameters.
</blockquote>

## SSH Keys

Coder generates SSH key pairs for each user. This can be used as an
authentication mechanism for git providers or other tools. Within workspaces,
git will attempt to use this key within workspaces via the `$GIT_SSH_COMMAND`
environment variable.

SSH keys are never stored in Coder workspaces, and are fetched only when
SSH is invoked. The keys are held in-memory and never written to disk.

Users can view their public key in their account settings:

![SSH keys in account settings](./images/ssh-keys.png)

## Dynamic Secrets

Dynamic secrets are attached to the workspace lifecycle and automatically
injected into the workspace. With a little bit of up front template work, they
make life simpler for both the end user and the security team.

This method is limited to
[services with Terraform providers](https://registry.terraform.io/browse/providers),
which excludes obscure API providers.

Add dynamic secrets into your template code using the following:

```hcl
resource "twilio_iam_api_key" "api_key" {
  account_sid   = "ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
  friendly_name = "Test API Key"
}

resource "coder_agent" "main" {
  # ...
  env = {
    # Let users access the secret via $TWILIO_API_SECRET
    TWILIO_API_SECRET = "${twilio_iam_api_key.api_key.secret}"
  }
}
```

A catch-all variation of this approach is dynamically provisioning a cloud
service account, for example 
[GCP](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/google_service_account_key#private_key),
for each workspace and then making the relevant secrets available via the
cloud's secret management system.

## Displaying Secrets

While you can inject secrets into the workspace via environment variables, you
can also show them in the Workspace UI with
[`coder_metadata`](https://registry.terraform.io/providers/coder/coder/latest/docs/resources/metadata).

![secret UI](./images/secret-metadata-ui.png)

Can be produced with

```hcl
resource "twilio_iam_api_key" "api_key" {
  account_sid   = "ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
  friendly_name = "Test API Key"
}


resource "coder_metadata" "twilio_key" {
  resource_id = twilio_iam_api_key.api_key.id
  item {
    key = "secret"
    value = twilio_iam_api_key.api_key.secret
    sensitive = true
  }
}
```
