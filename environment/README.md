<h1 align="center">Настройка среды разработки</h1>


<h2 align="center">Установка <code>Node.js</code></h2>

1. Установим `nvm`:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash

export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" 
```

2. Установим `Node.js`:

```bash
nvm install node 
nvm use node 
```

<h2 align="center">Установка и настройка <code>PostgreSQL</code></h2>

1. Установим `PostgreSQL`:

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

2. Настройка `PostgreSQL`:

Зададим пароль для пользователя `postgres`:
```bash
sudo -u postgres psql
ALTER USER postgres WITH PASSWORD 'новый_пароль';
```

Создадим базу данных с именем `secure_cloud_storage`
```
sudo -u postgres psql
CREATE DATABASE secure_cloud_storage;
```

<h2 align="center">Установка <code>React</code> (клиентская часть приложения)</h2>

1. Создадим директорию для клиентской части:

```bash
mkdir client
cd client
```

2. Инициализируем `React` приложение:

```bash
npx create-react-app .
```

3. Установим необходимые библиотеки:
```
npm install *название библиотеки*
```

<h2 align="center">Установка <code>Express</code> (серверная часть приложения)</h2>

1. Создадим директорию для серверной части:

```bash
mkdir server
cd server
```

2. Инициализируем `Node.js` проект:

```bash
npm init -y
```

3. Установим `Express`:

```bash
npm install express 
```

4. Установим необходимые библиотеки:
```
npm install *название библиотеки*
```