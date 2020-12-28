# Association

Creating associations in sequelize is done by calling one of the belongsTo, hasOne, hasMany, belongsToMany functions on a model (the source), and providing another model as the first argument to the function (the target).

- hasOne: adds a foreign key to the target and singular association mixins to the source.
- belongsTo: add a foreign key and singular association mixins to the source.
- hasMany: adds a foreign key to target and plural association mixins to the source.
- belongsToMany: creates an N:M association with a join table and adds plural association mixins to the source. The junction table is created with sourceId and targetId.

Creating an association will add a foreign key constraint to the attributes. All associations use CASCADE on update and SET NULL on delete, except for n:m, which also uses CASCADE on delete.

When creating associations, you can provide an alias, via the as option. This is useful if the same model is associated twice, or you want your association to be called something other than the name of the target model.

## Plain Explanation

When calling a method such as User.hasOne(Project), we say that the User model (the model that the function is being invoked on) is the source and the Project model (the model being passed as an argument) is the target.

### One-To-One associations

One-To-One associations are associations between exactly two models connected by a single foreign key.

#### BelongsTo

BelongsTo associations are associations where the foreign key for the one-to-one relation exists on the source model.

A simple example would be a Player being part of a Team with the foreign key on the player.

```javascript
var Player = this.sequelize.define("player", {
    /* attributes */
  }),
  Team = this.sequelize.define("team", {
    /* attributes */
  });

Player.belongsTo(Team); // Will add a teamId attribute to Player to hold the primary key value for Team
```

#### Foreign keys

By default the foreign key for a belongsTo relation will be generated from the target model name and the target primary key name.

The default casing is camelCase however if the source model is configured with underscored: true the foreignKey will be snake_case.

```javascript
var User = this.sequelize.define("user", {
    /* attributes */
  }),
  Company = this.sequelize.define("company", {
    /* attributes */
  });

User.belongsTo(Company); // Will add companyId to user

var User = this.sequelize.define(
    "user",
    {
      /* attributes */
    },
    { underscored: true }
  ),
  Company = this.sequelize.define("company", {
    uuid: {
      type: Sequelize.UUID,
      primaryKey: true,
    },
  });

User.belongsTo(Company); // Will add company_uuid to user
```

In cases where as has been defined it will be used in place of the target model name.

```javascript
var User = this.sequelize.define("user", {
    /* attributes */
  }),
  UserRole = this.sequelize.define("userRole", {
    /* attributes */
  });

User.belongsTo(UserRole, { as: "role" }); // Adds roleId to user rather than userRoleId
```

In all cases the default foreign key can be overwritten with the foreignKey option. When the foreign key option is used, Sequelize will use it as-is:

```javascript
var User = this.sequelize.define("user", {
    /* attributes */
  }),
  Company = this.sequelize.define("company", {
    /* attributes */
  });

User.belongsTo(Company, { foreignKey: "fk_company" }); // Adds fk_company to User
```

#### Target keys

The target key is the column on the target model that the foreign key column on the source model points to. By default the target key for a belongsTo relation will be the target model's primary key. To define a custom column, use the targetKey option.

```javascript
var User = this.sequelize.define("user", {
    /* attributes */
  }),
  Company = this.sequelize.define("company", {
    /* attributes */
  });

User.belongsTo(Company, { foreignKey: "fk_companyname", targetKey: "name" }); // Adds fk_companyname to User
```

#### HasOne

HasOne associations are associations where the foreign key for the one-to-one relation exists on the target model.

```javascript
var User = sequelize.define("user", {
  /* ... */
});
var Project = sequelize.define("project", {
  /* ... */
});

// One-way associations
Project.hasOne(User);

/*
In this example hasOne will add an attribute projectId to the User model!

Furthermore, Project.prototype will gain the methods getUser and setUser according to the first parameter passed to define. If you have underscore style enabled, the added attribute will be project_id instead of projectId.

The foreign key will be placed on the users table.

You can also define the foreign key, e.g. if you already have an existing database and want to work on it:
*/

Project.hasOne(User, { foreignKey: "initiator_id" });

/*
Because Sequelize will use the model's name (first parameter of define) for the accessor methods, it is also possible to pass a special option to hasOne:
*/

Project.hasOne(User, { as: "Initiator" });
// Now you will get Project.getInitiator and Project.setInitiator

// Or let's define some self references
var Person = sequelize.define("person", {
  /* ... */
});

Person.hasOne(Person, { as: "Father" });
// this will add the attribute FatherId to Person

// also possible:
Person.hasOne(Person, { as: "Father", foreignKey: "DadId" });
// this will add the attribute DadId to Person

// In both cases you will be able to do:
Person.setFather;
Person.getFather;

// If you need to join a table twice you can double join the same table
Team.hasOne(Game, { as: "HomeTeam", foreignKey: "homeTeamId" });
Team.hasOne(Game, { as: "AwayTeam", foreignKey: "awayTeamId" });

Game.belongsTo(Team);
```

Even though it is called a HasOne association, for most 1:1 relations you usually want the BelongsTo association since BelongsTo will add the foreignKey on the source where hasOne will add on the target.

#### Difference between HasOne and BelongsTo

In Sequelize 1:1 relationship can be set using HasOne and BelongsTo. They are suitable for different scenarios. Lets study this difference using an example.

Suppose we have two tables to link Player and Team. Lets define their models.

```javascript
var Player = this.sequelize.define("player", {
    /* attributes */
  }),
  Team = this.sequelize.define("team", {
    /* attributes */
  });
```

When we link two model in Sequelize we can refer them as pairs of source and target models. Like this:

- Player as the source
- Team as the target

```javascript
Player.belongsTo(Team);
//Or
Player.hasOne(Team);
```

Having Team as the source and Player as the target

```javascript
Team.belongsTo(Player);
//Or
Team.hasOne(Player);
```

HasOne and BelongsTo insert the association key in different models from each other. HasOne inserts the association key in target model whereas BelongsTo inserts the association key in the source model.

Here is an example demonstrating use cases of BelongsTo and HasOne.

```javascript
var Player = this.sequelize.define("player", {
    /* attributes */
  }),
  Coach = this.sequelize.define("coach", {
    /* attributes */
  }),
  Team = this.sequelize.define("team", {
    /* attributes */
  });
```

Suppose our Player model has information about its team as teamId column. Information about each Team's Coach is stored in the Team model as coachId column. These both scenarios requires different kind of 1:1 relation because foreign key relation is present on different models each time.

When information about association is present in source model we can use belongsTo. In this case Player is suitable for belongsTo because it has teamId column.

```javascript
Player.belongsTo(Team); // `teamId` will be added on Player / Source model
```

When information about association is present in target model we can use hasOne. In this case Coach is suitable for hasOne because Team model store information about its Coach as coachId field.

```javascript
Coach.hasOne(Team); // `coachId` will be added on Team / Target model
```

### One-To-Many associations

One-To-Many associations are connecting one source with multiple targets. The targets however are again connected to exactly one specific source.

```javascript
var User = sequelize.define("user", {
  /* ... */
});
var Project = sequelize.define("project", {
  /* ... */
});

// OK. Now things get more complicated (not really visible to the user :)).
// First let's define a hasMany association
Project.hasMany(User, { as: "Workers" });
```

This will add the attribute projectId or project_id to User. Instances of Project will get the accessors getWorkers and setWorkers. We could just leave it the way it is and let it be a one-way association. But we want more! Let's define it the other way around by creating a many to many association in the next section:

### Belongs-To-Many associations

Belongs-To-Many associations are used to connect sources with multiple targets. Furthermore the targets can also have connections to multiple sources.

```javascript
Project.belongsToMany(User, { through: "UserProject" });
User.belongsToMany(Project, { through: "UserProject" });
```

This will create a new model called UserProject with the equivalent foreign keys projectId and userId. Whether the attributes are camelcase or not depends on the two models joined by the table (in this case User and Project).

Defining through is required. Sequelize would previously attempt to autogenerate names but that would not always lead to the most logical setups.

This will add methods getUsers, setUsers, addUser,addUsers to Project, and getProjects, setProjects, addProject, and addProjects to User.

Sometimes you may want to rename your models when using them in associations. Let's define users as workers and projects as tasks by using the alias (as) option. We will also manually define the foreign keys to use:

```javascript
User.belongsToMany(Project, {
  as: "Tasks",
  through: "worker_tasks",
  foreignKey: "userId",
});
Project.belongsToMany(User, {
  as: "Workers",
  through: "worker_tasks",
  foreignKey: "projectId",
});
```

foreignKey will allow you to set source model key in the through relation. otherKey will allow you to set target model key in the through relation.

```javascript
User.belongsToMany(Project, {
  as: "Tasks",
  through: "worker_tasks",
  foreignKey: "userId",
  otherKey: "projectId",
});
```

Of course you can also define self references with belongsToMany:

```javascript
Person.belongsToMany(Person, { as: "Children", through: "PersonChildren" });
// This will create the table PersonChildren which stores the ids of the objects.
```

If you want additional attributes in your join table, you can define a model for the join table in sequelize, before you define the association, and then tell sequelize that it should use that model for joining, instead of creating a new one:

```javascript
User = sequelize.define("user", {});
Project = sequelize.define("project", {});
UserProjects = sequelize.define("userProjects", {
  status: DataTypes.STRING,
});

User.belongsToMany(Project, { through: UserProjects });
Project.belongsToMany(User, { through: UserProjects });
```

To add a new project to a user and set its status, you pass extra options.through to the setter, which contains the attributes for the join table

```javascript
user.addProject(project, { through: { status: "started" } });
```

By default the code above will add projectId and userId to the UserProjects table, and remove any previously defined primary key attribute - the table will be uniquely identified by the combination of the keys of the two tables, and there is no reason to have other PK columns. To enforce a primary key on the UserProjects model you can add it manually.

```javascript
UserProjects = sequelize.define("userProjects", {
  id: {
    type: Sequelize.INTEGER,
    primaryKey: true,
    autoIncrement: true,
  },
  status: DataTypes.STRING,
});
```

With Belongs-To-Many you can query based on through relation and select specific attributes. For example using findAll with through

```javascript
User.findAll({
  include: [
    {
      model: Project,
      through: {
        attributes: ["createdAt", "startedAt", "finishedAt"],
        where: { completed: true },
      },
    },
  ],
});
```

```javascript

```

```javascript

```

