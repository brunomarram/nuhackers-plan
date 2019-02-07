Comandos para gerenciar usuários e permissões mongoDB
---

**Criar nova regra**

```
use nucont //banco utilizado
db.createRole(
   {
     role: "manageOpRole",
     privileges: [
       { resource: { cluster: true }, actions: [ "killop", "inprog" ] },
       { resource: { db: "", collection: "" }, actions: [ "killCursors" ] }
     ],
     roles: []
   }
)
```

Para ver quais regras podem ser criadas [aqui](https://docs.mongodb.com/manual/tutorial/manage-users-and-roles/).


**Ver regras de um usuário**

```
use nucont
db.getUser("user")
```

**Para mostrar privilégios**

```
use nucont
db.getRole( "user", { showPrivileges: true } )
```

**Remover uma role do usuário**

```
use nucont
db.revokeRolesFromUser(
    "user",
    [
      { role: "readWrite", db: "nucont" }
    ]
)
```

**Adicionar uma regra ao usuário**

```
use nucont
db.grantRolesToUser(
    "user",
    [
      { role: "read", db: "nucont" }
    ]
)
```

**Alterar senha**

```
db.changeUserPassword("user", "SOh3TbYhxuLiW8ypJPxmt1oOfL")
```