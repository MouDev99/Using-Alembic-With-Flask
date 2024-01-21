Alembic is a great tool to use to manage the growth of your database. However, coupled with Flask through the Flask-Migrate and Flask-SQLAlchemy extensions, it is awesome!

In this article, you will get to see how awesome it is by

Configuring a Flask application to use Alembic;
Run commands to manage your database through the flask command; and,
Autogenerate migrations from your models!
That's right. Autogenerate. How great is that?

A little prework
If you're going to follow along with this article's code, create a PostgreSQL user named "flask_migrate_test" with the password "flask_migrate_test" and a database named (you guessed it!) "flask_migrate_test". Those will be the values used in this article's example code.

Configuring Flask to use Alembic
If you already have a Flask application with Flask-SQLAlchemy, SQLAlchemy and Psycopg2 (and a virtual environment that you've created and activated), then you can just use the following command to get Alembic and Flask-Migrate into your project.

pipenv install alembic Flask-Migrate
If you're starting from scratch, you can use this command. (If you're going to follow along, please use this command after creating a new directory and changing into it.)

pipenv install --python "$PYENV_ROOT/shims/python" psycopg2-binary \
               Flask-SQLAlchemy alembic Flask-Migrate Flask python-dotenv
That will install everything to make a happy database-driven Web application. (well, maybe not everything, especially if you dislike Jinja and love Pug.)

psycopg2-binary will allow SQLAlchemy to connect to you database
Flask-SQLAlchemy will integrate SQLAlchemy into your Flask application
alembic will manage the migrations for you
Flask-Migrate integrates Alembic with Flask
Flask is the Web server
python-dotenv allows you to put environment files in a .env or .flaskenv file that Flask will use
Once you have that, you will want to create some files to get everything working together. Then, it'll be time to run some commands.

First, create a .env or .flaskenv file, whichever you'd like, in the root of your project directory. In there, put this content.

FLASK_APP=flask_migrate_test.py
FLASK_ENV=development
DATABASE_URL=postgresql://flask_migrate_test:flask_migrate_test@localhost/flask_migrate_test
Set up the flask_migrate_test database user that the DATABASE_URL environment variable references. In a later section, you'll run a flask command to initialize the database.

CREATE USER flask_migrate_test WITH PASSWORD 'flask_migrate_test';
Now, create the following:

A directory name app
A file named app/__init__.py
A file named app/models.py
A file named flask_migrate_test.py in the project's root directory
The app/models.py file will contain the mapping classes for the application. For now, put the following content in app/models.py.

# app/models.py
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()
The app/__init__.py file will initialize the Flask application and bring everything together. It will create an instance of a Flask application, configure the SQLAlchemy object from app/models.py, and then configure the Flask-Migrate extension, too. To do that, put this content in the file. You can see setting the expected "SQLALCHEMY_DATABASE_URI" key to the value of the "DATABASE_URL" environment variable. (The "SQLALCHEMY_TRACK_MODIFICATIONS" is a deprecated feature that this code turns off so no warnings appear.)

# app/__init__.py
from app.models import db
from flask import Flask
from flask_migrate import Migrate
import os

app = Flask(__name__)
app.config.from_mapping({
  'SQLALCHEMY_DATABASE_URI': os.environ.get('DATABASE_URL'),
  'SQLALCHEMY_TRACK_MODIFICATIONS': False,
})
db.init_app(app)
Migrate(app, db)

Then, the only thing left to do is provide this to the flask runner. You do that by putting the following content in flask_migrate_test.py.

# flask_migrate_test.py
from app import app
All of this was so that you can experience the magic of Flask-Migrate.

Managing your database
Just like you had to initialize Alembic using the alembic init command, you need to do the same, here. However, the Flask-Migrate extension will generate a stronger, faster, smarter version of the Alembic environment.

The Flask-Migrate extension adds the db command to the flask command-line tool. To initialize the local Alembic environment (not the database), you type the following command.

pipenv run flask db init
After running that, you can see the layout of the code in the project.

.
├── Pipfile
├── Pipfile.lock
├── app
│   ├── __init__.py
│   └── models.py
├── flask_migrate_test.py
└── migrations
    ├── README
    ├── alembic.ini
    ├── env.py
    ├── script.py.mako
    └── versions
You can see that the Flask-Migrate extension has put all of the Alembic-specific stuff in the migrations directory for you! That's awesome!

Now, for the cool. Open up the app/models.py file and put a class in their for the Owner mapping like you did in the Flask-SQLAlchemy articles.

from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()


class Owner(db.Model):
    __tablename__ = "owners"

    id = db.Column(db.Integer, primary_key=True)
    first_name = db.Column(db.String(50), nullable=False)
    last_name = db.Column(db.String(50), nullable=False)
    email = db.Column(db.String(255), nullable=False)
Ok. Here it comes. Run the following command.

pipenv run flask db migrate -m "create owners table"
It created a file in the migrations/versions directory. Open it up and look at it. Do you see it?!?!

def upgrade():
    # ### commands auto generated by Alembic - please adjust! ###
    op.create_table('owners',
    sa.Column('id', sa.Integer(), nullable=False),
    sa.Column('first_name', sa.String(length=50), nullable=False),
    sa.Column('last_name', sa.String(length=50), nullable=False),
    sa.Column('email', sa.String(length=255), nullable=False),
    sa.PrimaryKeyConstraint('id')
    )
    # ### end Alembic commands ###


def downgrade():
    # ### commands auto generated by Alembic - please adjust! ###
    op.drop_table('owners')
    # ### end Alembic commands ###
It autogenerated the entire migration from your mapping file! Unlike Sequelize CLI, you write a model and it generates the migration for you!

Just to be clear: flask db migrate generates migrations. It does not apply them to your database. To do that, you use the upgrade command like you would with Alembic. But, unlike Alembic, if you don't specify a revision, it just applies all of them for you! Thanks, Flask-Migrate!

pipenv run flask db upgrade
And, you should see that the database has upgraded. You can confirm this by using Postbird or psql and looking that the "owners" table now exists!

The recipe is simple:

Add a model
Run pipenv run flask db migrate to autogenerate a migration
Run pipenv run flask db upgrade to apply it to the database
So easy!

All of the Alembic commands are there, too. If you type pipenv run flask db  --help, you'll see something like this.

Usage: flask db [OPTIONS] COMMAND [ARGS]...

  Perform database migrations.

Options:
  --help  Show this message and exit.

Commands:
  branches   Show current branch points
  current    Display the current revision for each database.
  downgrade  Revert to a previous version
  edit       Edit a revision file
  heads      Show current available heads in the script directory
  history    List changeset scripts in chronological order.
  init       Creates a new migration repository.
  merge      Merge two revisions together, creating a new revision file
  migrate    Autogenerate a new revision file (Alias for 'revision...
  revision   Create a new revision file.
  show       Show the revision denoted by the given symbol.
  stamp      'stamp' the revision table with the given revision; don't run...
  upgrade    Upgrade to a later version
All of the Alembic commands are there, like merge and history. You just need to use flask db instead of alembic.
