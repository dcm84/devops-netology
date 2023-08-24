# devops-netology is now modified

## terraform/.gitignore
исключает из коммитов:
- содержимое папок .terraform/
- файлы, содержащие .tfstate в названии
- файлы crash.log и файлы, начинающиеся с crash. и заканчивающиеся на .log
- файлы, оканчивающиеся на .tfvars и .tfvars.json
- override.tf, override.tf.json и файлы, оканчивающиеся на _override.tf и _override.tf.json
- файлы .terraformrc и terraform.rc

## committed via vscode