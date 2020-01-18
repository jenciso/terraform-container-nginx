# Terraform LAB 101

# Deploy NGINX container using Terraform

## Create Terraform Config

Terraform works based on a configuration file, in this case config.tf. The configuration defines your infrastructure, in this instance as providers and resources.

A provider is an abstract way of handling the underlying infrastructure and responsible for managing the lifecycle of a resource.

A resource are components of your infrastructure, for example a container or image.

We will use a `config.tf` file to setup our lab environment.

```shell
cat > config.tf << EOF
provider "docker" {
  host = "unix:///var/run/docker.sock"
}
EOF
```

We can now start defining the resources of our infrastructure. The first resource is our Docker image. A resource has two parameters, one is a `TYPE` and second a `NAME`. The type is `docker_image` and the name is `nginx`. Within the block we define the name and tag of the Docker Image.

```shell
cat >> config.tf << EOF
resource "docker_image" "nginx" {
  name = "nginx:1.11-alpine"
}
EOF
```

We can define our container resource. The resource type is `docker_container` and name as `nginx-server`. Within the block we set the resource parameters. We can reference other resources, such as a the `image`.

```shell
cat >> config.tf << EOF
resource "docker_container" "nginx-server" {
  name = "nginx-server-1"
  image = docker_image.nginx.latest
  ports {
    internal = 80
    external = 8081
  }
  volumes {
    container_path  = "/usr/share/nginx/html"
    host_path = "/tmp/tutorial/www"
    read_only = true
  }
}
EOF
```

## Plan Terraform Actions

Once the configuration has been defined we need to create an execution plan. Terraform describes the actions required to achieve the desired state. The plan can be saved using -out. We'll apply the execution plan in the next step.

First at all:

```shell
terraform init
```

To create a plan, use the CLI

```shell
terraform plan -out config.tfplan
```

The output of the command indicates the changes. In this case, you'll see a _dockercontainer.nginx-server_ and _dockerimage.nginx_ to highlight adding the new resources.

Finally a summary of Plan: `2 to add, 0 to change, 0 to destroy`.

## Adding some content

We could add a some content to serve via our nginx server.

```shell
mkdir -p /tmp/tutorial/www
echo "<h1>hello world</h1>" > /tmp/tutorial/www/index.html
```

## Apply Terraform Actions

Once the plan has been created we need to apply it to reach our desired state.

Using the CLI, terraform will pull any images required and launch new containers.


```shell
terraform apply
```

## Inspecting Infrastructure

You can use the Docker CLI to view the changes and see the newly launched container.

```shell
docker ps
```

you can inspect this in future using the terraform CLI

```shell
terraform show
```

## Testing

```shell
curl http://localhost:8081
firefox http://localhost:8081
```

## Updating Infrastructure

As our infrastructure grows and changes, terraform will manage and ensure we always have our defined desired state.

We can change our container to launch two instances, each with different names.

```ruby
resource "docker_container" "nginx-server" {
  count = 2
  name = "nginx-server-${count.index+1}"
  image = docker_image.nginx.latest
  ports {
    internal = 80
    external = "808${count.index+1}"
  }
  volumes {
    container_path  = "/usr/share/nginx/html"
    host_path = "/tmp/tutorial/www"
    read_only = true
  }
}
```

If we create a plan you will see the actions Terraform will need to apply to adapt our infrastructure to match our configuration.

```shell
terraform validate
terraform plan -out config.tfplan
```

The plan will outline the changes. Because we're changing the name and adding a resource we'll see `Plan: 1 to add, 0 to change, 0 to destroy.`

In the details it will explain that changing a container name forces the resource to be recreated name: "nginx-server" => "nginx-server-1" (forces new resource) along with adding the new container _dockercontainer.nginx-server.1_

We can then apply the plan as we did in the previous step.

```shell
terraform apply -auto-approve
```

Now, scale your nginx-server to 8 replicas:

```ruby
resource "docker_container" "nginx-server" {
  count = 8
  name = "nginx-server-${count.index+1}"
  image = docker_image.nginx.latest
  ports {
    internal = 80
    external = "808${count.index+1}"
  }
  volumes {
    container_path  = "/usr/share/nginx/html"
    host_path = "/tmp/tutorial/www"
    read_only = true
  }
}
```

```shell
terraform plan
terraform apply -auto-approve
```

## Clean UP

To destroy all the resources created

```shell
terraform destroy
```

To delete all files created
```
rm -rf config.tf* terraform.tfstate* .terraform/
rm -rf /tmp/tutorial
```

### DEMO

[![asciicast](https://asciinema.org/a/bF5OfbZwLZ043XvJ0au3ATPPV.svg)](https://asciinema.org/a/bF5OfbZwLZ043XvJ0au3ATPPV)
