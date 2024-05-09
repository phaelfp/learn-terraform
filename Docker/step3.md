# Destroy infrastructure

On this page:

 - [Destroy](#destroy)
 - [Next steps](#next-steps)

 You have now created and updated a Docker container on your machine with Terraform. In this tutorial, you will use Terraform to destroy this infrastructure.

Once you no longer need infrastructure, you may want to destroy it to reduce your security exposure, costs, or resource overhead. For example, you may remove a production environment from service, or manage short-lived environments like build or testing systems. In addition to building and modifying infrastructure, Terraform can destroy or recreate the infrastructure it manages.

# Destroy
The `terraform destroy` command terminates resources managed by your Terraform project. This command is the inverse of `terraform apply` in that it terminates all the resources specified in your Terraform state. It does not destroy resources running elsewhere that are not managed by the current Terraform project.

Destroy the resources you created.

```bash
terraform destroy
docker_image.nginx: Refreshing state... [id=sha256:d1a364dc548d5357f0da3268c888e1971bbdb957ee3f028fe7194f1d61c6fdeenginx:latest]
docker_container.nginx: Refreshing state... [id=b2140f8c6aa79f62c8ac3c3d792f2044bcca8d5a0a08a4598ead1ade7aab7e6e]

Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

   docker_container.nginx will be destroyed
  - resource "docker_container" "nginx" {
#...

   docker_image.nginx will be destroyed
  - resource "docker_image" "nginx" {
      - id           = "sha256:d1a364dc548d5357f0da3268c888e1971bbdb957ee3f028fe7194f1d61c6fdeenginx:latest" -> null
      - keep_locally = false -> null
      - latest       = "sha256:d1a364dc548d5357f0da3268c888e1971bbdb957ee3f028fe7194f1d61c6fdee" -> null
      - name         = "nginx:latest" -> null
    }

Plan: 0 to add, 0 to change, 2 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value:
```

The `-` prefix indicates that the container will be destroyed. As with apply, Terraform shows its execution plan and waits for approval before making any changes.

Answer `yes` to execute this plan and destroy the infrastructure.

```bash
  Enter a value: yes

docker_container.nginx: Destroying... [id=b2140f8c6aa79f62c8ac3c3d792f2044bcca8d5a0a08a4598ead1ade7aab7e6e]
docker_container.nginx: Destruction complete after 2s
docker_image.nginx: Destroying... [id=sha256:d1a364dc548d5357f0da3268c888e1971bbdb957ee3f028fe7194f1d61c6fdeenginx:latest]
docker_image.nginx: Destruction complete after 0s

Destroy complete! Resources: 2 destroyed.
```

Just like with `apply`, Terraform determines the order to destroy your resources. In this case, Terraform identified a single container with no other dependencies, so it destroyed the container. In more complicated cases with multiple resources, Terraform will destroy them in a suitable order to respect dependencies.

# Next steps
Working with [Variables](step4.md)