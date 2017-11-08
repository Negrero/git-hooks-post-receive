# Remote server
* Creamos el directorio para nuestro cogido de despliegue
```
ssh user@server.com
mkdir ~/deploy
```
* Añadimos un repositorio limpio de git
```
$ git init --bare ~/git-hooks-post-receive.git
```
* Añadimos el hook post-receive en git
```
cd ~/git-hooks-post-receive.git/hooks
nano post-receive
chmod +x post-receive
```
Contenido del fichero post-receive:
```
#!/bin/bash
while read oldrev newrev ref
do
    # only checking out the master (or whatever branch you would like to deploy)
    if [[ $ref =~ .*/production$ ]];
    then
        echo "Master ref received.  Deploying master branch to production..."
        git --work-tree=/home/ubuntu/deploy --git-dir=/home/ubuntu/git-hooks-post-receive.git checkout -f production
		cd /home/ubuntu/deploy
		slc build
		slc deploy http://localhost:8701 ../git-hooks-post-receive-0.0.1.tgz
    else
        echo "Ref $ref successfully received.  Doing nothing: only the production branch may be deployed on this server."
    fi
done
```
Otra alternativa del fichero post-receive:
```
#!/bin/bash

target_branch="production"
working_tree="deploy"

while read oldrev newrev refname
do
    branch=$(git rev-parse --symbolic --abbrev-ref $refname)
    if [ -n "$branch" ] && [ "$target_branch" == "$branch" ]; then
    
       GIT_WORK_TREE=$working_tree git checkout $target_branch -f
       NOW=$(date +"%Y%m%d-%H%M")
       git tag release_$NOW $target_branch
    
       echo "   /==============================="
       echo "   | DEPLOYMENT COMPLETED"
       echo "   | Target branch: $target_branch"
       echo "   | Target folder: $working_tree"
       echo "   | Tag name     : release_$NOW"
       echo "   \=============================="
       slc build
	   slc deploy http://localhost:8701 ../git-hooks-post-receive-0.0.1.tgz
    fi
done
```
### Nota
Fijemonos en la linea:
```
  git --work-tree=/home/ubuntu/deploy --git-dir=/home/ubuntu/git-hooks-post-receive.git checkout -f production
```
donde:
* --work-tree será el directorio donde estará nuestro código a desplegar
* --git-dir será el directorio donde estará el las carpetas propias de git

# Local

* Creamos un proyecto
```
git init git-hooks-post-receive
cd git-hooks-post-receive
npm init
npm install express --save
```
* Editamos el fichero index.js
```
var express = require('express');
var app = express();

app.get('/', function (req, res) {
  res.send('Hello World!porfin un pasito massssssssssssss');
});

app.listen(3000, function () {
  console.log('Example app listening on port 3000');
});

```
* Creamos la rama production y mezclamos con master
```
git branch -a production
git merge master
```
* Conectar con server el git local
```
git add remote deploy_production ubuntu@192.168.0.100:git-hooks-post-receive.git
```
* Empujar al servidor produccion

```
git push deploy_production
```

### Nota
La conexion con el servidor será con ssh por lo que el servidor tendrá al menos un servidor que permita conexiones ssh para la comunicación con git del servidor.
También puntualizar que hay que descartar de la url de conexion("ubuntu@192.168.0.100:git-hooks-post-receive.git") el "/home/ubuntu" ya que por defecto la conexion ssh nos situará en el directorio de trabajo del usuario con el que vamos a conectar.
