# Задача 1
## 1.1. Скачайте зависимости
```
terraform init

Initializing the backend...

Initializing provider plugins...
- Finding kreuzwerker/docker versions matching "~> 3.0.1"...
- Finding latest version of hashicorp/random...
- Installing kreuzwerker/docker v3.0.2...
- Installed kreuzwerker/docker v3.0.2 (self-signed, key ID BD080C4571C6104C)
- Installing hashicorp/random v3.6.0...
- Installed hashicorp/random v3.6.0 (signed by HashiCorp)
...
Terraform has been successfully initialized!
...
```

## 1.2. Где хранить секреты
Согласно  .gitignore секреты предполагается хранить в personal.auto.tfvars

## 1.3. Достать из state-файла random_password
```
..
"resources": [
    {
      ...
      "type": "random_password",
      "name": "random_string",
      "provider": "provider[\"registry.terraform.io/hashicorp/random\"]",
      "instances": [
        {
          ...
          "attributes": {
            ...
            "result": "JtWRPfrfewy0vTXG",
            ...
          },
          ...
        }
      ]
    }
  ]
```

## 1.4. Объясните, в чём заключаются намеренно допущенные ошибки
### 1.4.1.
```
│ Error: Missing name for resource
│ 
│   on main.tf line 24, in resource "docker_image":
│   24: resource "docker_image" {
│ 
│ All resource blocks must have 2 labels (type, name).
```
пропустили название ресурса

### 1.4.2.
```
│ Error: Invalid resource name
│ 
│   on main.tf line 29, in resource "docker_container" "1nginx":
│   29: resource "docker_container" "1nginx" {
│ 
│ A name must start with a letter or underscore and may contain only letters, digits, underscores, and dashes.
```
имена надо начинать с букв и _

### 1.4.3.
```
│ Error: Reference to undeclared resource
│ 
│   on main.tf line 31, in resource "docker_container" "nginx1":
│   31:   name  = "example_${random_password.random_string_FAKE.resulT}"
│ 
│ A managed resource "random_password" "random_string_FAKE" has not been declared in the root module.
```
ссылаемся на несуществующий ресурс

### 1.4.4.
```
│ Error: Unsupported attribute
│ 
│   on main.tf line 31, in resource "docker_container" "nginx1":
│   31:   name  = "example_${random_password.random_string.resulT}"
│ 
│ This object has no argument, nested block, or exported attribute named "resulT". Did you mean "result"?
```
ссылаемся на неправильный атрибут ресурса

## 1.5. Выполните код
Исправленный фрагмент
```
...
resource "docker_image" "nginx"{
  name         = "nginx:latest"
  keep_locally = true
}

resource "docker_container" "nginx1" {
  image = docker_image.nginx.image_id
  name  = "example_${random_password.random_string.result}"

  ports {
    internal = 80
    external = 8000
  }
}

```

Применяем изменения ``terraform apply``
```
terraform-01$ terraform apply
...

docker_image.nginx: Creating...
docker_image.nginx: Still creating... [10s elapsed]
docker_image.nginx: Creation complete after 11s [id=sha256:d453dd892d9357f3559b967478ae9cbc417b52de66b53142f6c16c8a275486b9nginx:latest]
docker_container.nginx1: Creating...
docker_container.nginx1: Creation complete after 2s [id=d4fd95511b1c339d6c672ce5b4cd217c3c6f52cde12011caf3c7b286ce139915]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

```

Вывод команды ``docker ps``
```
terraform-01$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                  NAMES
d4fd95511b1c   d453dd892d93   "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   0.0.0.0:8000->80/tcp   example_JtWRPfrfewy0vTXG
```

## 1.6. Заменить имя docker-контейнера на hello_world
Выполните команду ``terraform apply -auto-approve``
```
terraform apply -auto-approve
...

Terraform will perform the following actions:

  # docker_container.hello_world will be created
  ...
  # docker_container.nginx1 will be destroyed
  ...
Plan: 1 to add, 0 to change, 1 to destroy.
docker_container.nginx1: Destroying... [id=d4fd95511b1c339d6c672ce5b4cd217c3c6f52cde12011caf3c7b286ce139915]
docker_container.hello_world: Creating...
docker_container.nginx1: Destruction complete after 0s
╷
│ Error: Unable to create container: Error response from daemon: Conflict. The container name "/example_JtWRPfrfewy0vTXG" is already in use by container "d4fd95511b1c339d6c672ce5b4cd217c3c6f52cde12011caf3c7b286ce139915". You have to remove (or rename) that container to be able to reuse that name.
│ 
│   with docker_container.hello_world,
│   on main.tf line 29, in resource "docker_container" "hello_world":
│   29: resource "docker_container" "hello_world" {
```
Ключ -auto-approve позволяет автоматически применить план без дополнительных подтверждений со стороны пользователя. В рекомендациях написано: **If you use -auto-approve, we recommend making sure that no one can change your infrastructure outside of your Terraform workflow. This minimizes the risk of unpredictable changes and configuration drift.** Ключ может пригодиться, если мы хотим выполнять terraform apply из скриптов (в пайплайнах, например).

В нашем случае создание и удаление контейнеров происходило параллельно, и новый контейнер не поднялся, так как контейнер с таким названием еще существовал.

```
terraform-01$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

## 1.7. Уничтожьте созданные ресурсы с помощью terraform
```
terraform-01$ terraform destroy
docker_image.nginx: Refreshing state... [id=sha256:d453dd892d9357f3559b967478ae9cbc417b52de66b53142f6c16c8a275486b9nginx:latest]
random_password.random_string: Refreshing state... [id=none]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:# Задача 1
## 1.1. Скачайте зависимости
```
terraform init

Initializing the backend...

Initializing provider plugins...
- Finding kreuzwerker/docker versions matching "~> 3.0.1"...
- Finding latest version of hashicorp/random...
- Installing kreuzwerker/docker v3.0.2...
- Installed kreuzwerker/docker v3.0.2 (self-signed, key ID BD080C4571C6104C)
- Installing hashicorp/random v3.6.0...
- Installed hashicorp/random v3.6.0 (signed by HashiCorp)
...
Terraform has been successfully initialized!
...
```
  # random_password.random_string will be destroyed
  ...
Plan: 0 to add, 0 to change, 2 to destroy.

...

docker_image.nginx: Destroying... [id=sha256:d453dd892d9357f3559b967478ae9cbc417b52de66b53142f6c16c8a275486b9nginx:latest]
docker_image.nginx: Destruction complete after 0s
random_password.random_string: Destroying... [id=none]
random_password.random_string: Destruction complete after 0s

Destroy complete! Resources: 2 destroyed.
```

terraform.tfstate:
```
{
  "version": 4,
  "terraform_version": "1.6.0",
  "serial": 10,
  "lineage": "092f5146-117c-5de4-ca52-fe3c4038cece",
  "outputs": {},
  "resources": [],
  "check_results": null
}
```

## 1.8. Объясните, почему при этом не был удалён docker-образ nginx:latest.
Образ не был удален, так как при объявлении ресурса docker_image используется ключ ``keep_locally=true``.
```
keep_locally (Boolean) If true, then the Docker image won't be deleted on destroy operation. If this is false, it will delete the image from the docker local storage on destroy operation.
```


# Задача 2
Создайте любую виртуальную машину

```
terraform-01/task2$ terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # virtualbox_vm.vm1 will be created
  + resource "virtualbox_vm" "vm1" {
      + cpus   = 1
      + id     = (known after apply)
      + image  = "https://app.vagrantup.com/shekeriev/boxes/debian-11/versions/0.2/providers/virtualbox.box"
      + memory = "512 mib"
      + name   = "debian-11"
      + status = "running"

      + network_adapter {
          + device                 = "IntelPro1000MTDesktop"
          + host_interface         = "vboxnet1"
          + ipv4_address           = (known after apply)
          + ipv4_address_available = (known after apply)
          + mac_address            = (known after apply)
          + status                 = (known after apply)
          + type                   = "hostonly"
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + IPAddress = (known after apply)
```