# KeyCloak and Discourse Docker dev environment
* KeyCloak URL: http://localhost:8080
* Discourse URL: http://localhost:3000

### 1. Clone Discourse 
```bash
$ git clone https://github.com/discourse/discourse.git
```

Modify `discourse/config/database.yml` to add `username: discourse` to the development database connection configuration:

```yml
development:
  prepared_statements: false
  username: discourse
  ...
```

### 2. Start the containers
```bash
$ docker-compose up
```

### 3. Set-up Discourse
```bash
$ docker-compose exec -w /src discourse bundle install
$ docker-compose exec -w /src discourse yarn install
$ docker-compose exec -w /src discourse rake db:migrate
$ docker-compose exec -w /src discourse rake admin:create
Email:
...
```
Complete the email and password and set the user to be an admin.

### 4. Start Discourse server
```bash
$ docker-compose exec -w /src discourse rails server
I, [2024-01-25T14:25:22.938512 #5719]  INFO -- : Refreshing Gem list
Starting CSS change watcher
I, [2024-01-25T14:25:25.889439 #5719]  INFO -- : listening on addr=0.0.0.0:3000 fd=24
...
```

