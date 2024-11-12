# <p style="text-align:center;"> **Домашнее задание к занятию «Что такое DevOps. СI/СD»  <Фамилия Имя>** </p>

## Задание №1

**1. Установка Jenkins**

```
- Обновление пакетов и установка java
apt update && apt install -y jenkins

- Устанавливаем Java
apt install fontconfig openjdk-17-jre


- Устанавливаем зависимый пакет
apt install gnupg2 -y

- Запускаем скрипт для настройки репозитория Jenkis
sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
```
![Установленная Java](https://github.com/HartLAS/8-2-hw/pictures/java_installed.png)

![Установленный Jenkins](https://github.com/HartLAS/8-2-hw/pictures/jenkins_installed.png)

![Jenkins UI](https://github.com/HartLAS/8-2-hw/pictures/jenkins_ui.png)
 
 **2. Установка Go**
 
```
- Скачиваем и распаковываем архив
wget https://go.dev/dl/go1.23.3.linux-amd64.tar.gz &&  && tar -zxf go1.23.3.linux-amd64.tar.gz

- Добавляем переменную среды
export PATH=$PATH:/usr/local/go/bin

- Переносим исполняемые файлы
mv go /usr/local/

- Проверяем установку
go version
```
 
![Установленный Golang](https://github.com/HartLAS/8-2-hw/pictures/go_installed.png)

 **3. Установка Docker**
 
```
- Устанавливаем зависимые пакеты
apt-get install ca-certificates curl

- Задаем права доступа на каталог ключей
install -m 0755 -d /etc/apt/keyrings

- Скачиваем ключ
curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc

- Задаем права на скачанный ключ
chmod a+r /etc/apt/keyrings/docker.asc

- Добавляем репозиторий докера
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null
  
- Обновляем список пакетов
apt-get update

- Уставнавливаем докер
apt-get install docker-ce docker-ce-cli containerd.io

- Запускаем сервис докера
service docker start
```

**4. Настраиваем проект в Jenkins**

- Натравливаем на GitHub
    - Ставим галку на `GitHub Project` и в поле вставляем `https://github.com/netology-code/sdvps-materials.git/`
    - Опускаемся до `Управление исходным кодом`, выбираем `Git`, вставляем в поле `Управление исходным кодом`
- Описываем задачи
    - Опускаемся до `Шаги сборки`
    - Выбираем `Выполнить команду shell`
    - Вставляем код из `https://github.com/netology-code/sdvps-materials/blob/main/CICD/8.2-hw.md`
```
    /usr/local/go/bin/go test .
    sudo docker build . 
```

![Настройки Jenkins](https://github.com/HartLAS/8-2-hw/pictures/jenkins_settings_1.png)

![Настройки Jenkins](https://github.com/HartLAS/8-2-hw/pictures/jenkins_settings_2.png)

![Настройки Jenkins](https://github.com/HartLAS/8-2-hw/pictures/jenkins_settings_3.png)

![Успешный пайп скрин №1](https://github.com/HartLAS/8-2-hw/pictures/success_pipe.png)

![Успешный пайп скрин №2](https://github.com/HartLAS/8-2-hw/pictures/success_pipe_end.png)

## Задание №2

- Создаем новый `Item` и выбираем 'PipeLine'
- Ставим галку на `GitHub Project` и в поле вставляем `https://github.com/netology-code/sdvps-materials.git/`
- В поле `PipeLine` вставляем код:
```
        pipeline {
     agent any
     stages {
      stage('Git') {
       steps {git 'https://github.com/netology-code/sdvps-materials.git'}
      }
      stage('Test') {
       steps {
        sh '/usr/local/go/bin/go test .'
       }
      }
      stage('Build') {
       steps {
        sh 'docker build . -t ubuntu-bionic:8082/hello-world:v$BUILD_NUMBER'
       }
      }
     }
    }
```

![Настройки Jenkins](https://github.com/HartLAS/8-2-hw/pictures/pipeline_settings_1.png)

![Настройки Jenkins](https://github.com/HartLAS/8-2-hw/pictures/pipeline_settings_2.png)

![Успешный пайп скрин №1](https://github.com/HartLAS/8-2-hw/pictures/pipeline_success_1.png)

![Успешный пайп скрин №2](https://github.com/HartLAS/8-2-hw/pictures/pipeline_success_2.png)

## Задание №3

- Устанавливаем `Nexus`
```
     docker run -d -p 8081:8081 -p 8082:8082 --name nexus -e INSTALL4J_ADD_VM_PARAMS="-Xms512m -Xmx512m -XX:MaxDirectMemorySize=273m" sonatype/nexus3
```
- Заходим на http://192.168.1.92:8081/ и авторизуемся под `admin`
- Создаем raw-hosted репозиторий
- Изменяем код пайплайна на 
```
pipeline {
 agent any
 stages {
  stage('Git') {
   steps {git 'https://github.com/netology-code/sdvps-materials.git'}
  }
  stage('Test') {
   steps {
    sh '/usr/local/go/bin/go test .'
   }
  }
  stage('Build') {
   steps {
    sh '''
    /usr/local/go/bin/go build -a -installsuffix nocgo -o test
    curl -v --user 'admin:80f2e5fe-1b18-4134-9541-6a524c23b1bb' --upload-file ./test http://localhost:8081/repository/netology/test
    '''
   }
  }
 }
}
```

![Репозиторий Nexus](https://github.com/HartLAS/8-2-hw/pictures/nexus-repo.png)

![Успешный пайп](https://github.com/HartLAS/8-2-hw/pictures/task_3_1.png)

![Успешный пайп](https://github.com/HartLAS/8-2-hw/pictures/task_3_2.png)

## Задание №4

- Меняем код пайпа на 
```
pipeline {
 agent any
 stages {
  stage('Git') {
   steps {git 'https://github.com/netology-code/sdvps-materials.git'}
  }
  stage('Test') {
   steps {
    sh '/usr/local/go/bin/go test .'
   }
  }
  stage('Build') {
   steps {
    sh '''
    /usr/local/go/bin/go build -a -installsuffix nocgo -o $BUILD_NUMBER
    curl -v --user 'admin:80f2e5fe-1b18-4134-9541-6a524c23b1bb' --upload-file ./$BUILD_NUMBER http://localhost:8081/repository/netology/$BUILD_NUMBER
    '''
   }
  }
 }
}
```

![Успешный пайп](https://github.com/HartLAS/8-2-hw/pictures/task_4_1.png)

![Успешный пайп](https://github.com/HartLAS/8-2-hw/pictures/task_4_2.png)