# Your First Homelab: A Beginner's Guide

Demo files and slides for the talk **"Your first homelab: a beginner's guide to DevOps and Data Engineering"**, presented at **AWS Student Community Day Kampala** (ADB Building, MUBS, Saturday 7 November 2026) by Jacob Kipsang.

No experience needed. In five small steps you go from zero to a running website, a database, and a chart, all on your own laptop with Docker.

```
docker run hello-world
```

## What's in this repo

| Path | What it is |
| --- | --- |
| `homelab-demo/compose.yml` | The whole live demo in one Docker Compose file. Each step is a commented-out block you switch on one at a time. |
| `homelab-demo/site/index.html` | The web page served in Step 2. Edit it and make it yours. |
| `homelab-demo/Your First Homelab A Beginner's Guide.pdf` | Slides (PDF). |
| `homelab-demo/Your First Homelab A Beginner's Guide.pptx` | Slides (PowerPoint). |

## What you need

- **A computer**: any laptop with 4 GB of RAM or more. An old one is perfect.
- **Free software**: Ubuntu Linux and Docker (with Docker Compose).
- **A little time**: about an hour a day for a week.

**On Windows 11?** Run `wsl --install` to get Ubuntu inside Windows, then install Docker Desktop.
**On Windows 10 or older?** Install VirtualBox, run Ubuntu inside it, and install Docker there.

Check Docker is working:

```bash
docker run hello-world
docker compose version
```

## Get started

```bash
git clone https://github.com/jkipsang/aws_std_community_day_homelab.git
cd aws_std_community_day_homelab/homelab-demo
```

### How the demo file works

`compose.yml` is one file with five steps. For each step:

1. **Uncomment** the step: delete the `# ` at the start of its lines. Lines starting with `##` are notes, so leave those alone.
2. **Run** `docker compose up -d`
3. **Check** `docker compose ps`

## The five steps

### Step 1 · DevOps: run your first containers

Uncomment the `hello` and `web` services, then:

```bash
docker compose up -d
docker compose logs hello   # shows "Hello from Docker!"
docker compose ps           # lists your running containers
```

Open <http://localhost:8080> to see the nginx welcome page.

### Step 2 · DevOps: put your own web page online

Uncomment the `mysite` service. It serves the `site/` folder with nginx:

```yaml
mysite:
  image: nginx
  ports:
    - "8081:80"
  volumes:
    - ./site:/usr/share/nginx/html
```

```bash
docker compose up -d
```

Open <http://localhost:8081> to see your page from `site/index.html`. Edit the file, refresh the browser, and it changes instantly. That's DevOps in miniature.

### Step 3 · Data: start a database

Uncomment the `db` service **and** the `volumes:` block at the bottom of the file, then:

```bash
docker compose up -d
docker compose exec db psql -U postgres
```

A full PostgreSQL 16 database starts in seconds. When the prompt changes to `postgres=#`, you're inside it.

### Step 4 · Data: store data and ask it questions

No new container. Inside `psql` from Step 3, type:

```sql
CREATE TABLE marks (name TEXT, course TEXT, score INT);

INSERT INTO marks VALUES
  ('Amina','Cloud',78), ('Brian','Cloud',65),
  ('Grace','Data',88),  ('Peter','Data',72);

SELECT course, ROUND(AVG(score), 1) AS avg
FROM marks GROUP BY course;
```

You should see:

```
 course | avg
--------+------
 Cloud  | 71.5
 Data   | 80.0
```

Leave `psql` with `\q`.

### Step 5 · Data: chart it with Metabase

Uncomment the `metabase` service, then:

```bash
docker compose up -d
```

Wait about a minute, then open <http://localhost:3000> and:

1. Add a database: **PostgreSQL**, host `db`, port `5432`, database `postgres`, user `postgres`, password `pass`.
2. **New → Question → marks**
3. Summarize **Average of score** by **course**
4. **Visualize → Bar**

You don't need any network commands: Compose puts every service on one network, so Metabase finds the database by the name `db`.

**The result:** raw marks became a chart. Extract → Transform → Load → Visualize. Four containers and a few commands give you a tiny version of what data engineers do every day.

## Clean up

```bash
docker compose down      # stop and remove the containers
docker compose down -v   # also delete the database data
```

## Ports used

| Service | URL |
| --- | --- |
| nginx welcome page (Step 1) | <http://localhost:8080> |
| Your own site (Step 2) | <http://localhost:8081> |
| PostgreSQL (Step 3) | `localhost:5432` |
| Metabase (Step 5) | <http://localhost:3000> |

## Troubleshooting

- **"port is already allocated"**: another container or app is using that port. Run `docker compose ps` or `docker ps` to find it, or change the left-hand number in `ports:` (for example `"8082:80"`).
- **Metabase page won't load**: it takes about a minute to start the first time. Check progress with `docker compose logs -f metabase`.
- **Metabase can't connect to the database**: use host `db`, not `localhost`, because Metabase is talking to it from inside the Compose network.
- **Started over and your table is gone**: `docker compose down -v` deletes the `pgdata` volume. Leave off `-v` to keep your data.

## Homelab → AWS

Everything you ran here has a managed equivalent on AWS.

**DevOps & infrastructure**

| Homelab | On AWS |
| --- | --- |
| Laptop + Ubuntu | Amazon EC2 |
| Docker + `compose.yml` | Amazon ECS + Fargate |
| Docker images | Amazon ECR |
| nginx + `site/` | Amazon S3 + CloudFront |
| `pgdata` volume | Amazon EBS |
| `compose logs` / `ps` | Amazon CloudWatch |
| Compose network | Amazon VPC |

**Data engineering**

| Homelab | On AWS |
| --- | --- |
| CSV / raw files | Amazon S3 (data lake) |
| ETL pipeline (Apache NiFi, Airbyte, dbt) | AWS Glue |
| PostgreSQL (`db`) | Amazon RDS |
| Big data warehouse (ClickHouse, Apache Hive) | Amazon Redshift |
| SQL on files | Amazon Athena |
| cron / Apache Airflow | Amazon MWAA |
| Metabase | Amazon QuickSight |

## Your 7-day beginner plan

| Day | Goal |
| --- | --- |
| 1 | **Set up Linux.** Windows 11: `wsl --install`. Windows 10 or older: Ubuntu in VirtualBox. |
| 2 | **Learn the terminal.** Practise `ls`, `cd`, `mkdir`, `cp` and `nano` until they feel normal. |
| 3 | **Install Docker.** Run `hello-world`, your first container. |
| 4 | **DevOps: run a website.** Start nginx and change its home page to say your name. |
| 5 | **Data: run a database.** Start PostgreSQL in Docker and create your first table. |
| 6 | **Data: make a chart.** Load a CSV of your choice and chart it in Metabase. |
| 7 | **Share it.** Write up what you learned and put your notes on GitHub. |
| After | **Level up.** Try automatic deployments, monitoring and data pipelines. |

## Four beginner rules

1. **Breaking it is normal.** If something breaks, you're learning. You can always reinstall and start again.
2. **Search the error.** Paste the error message into a search engine or an AI assistant. Someone has seen it before.
3. **Write it down.** Keep every command you run in a notes file. It becomes your portfolio later.
4. **Keep it private.** Don't open your homelab to the internet yet. Learn on your own network first. (The password `pass` in this demo is for local practice only.)

## Free places to learn

| Topic | Resource | Where |
| --- | --- | --- |
| Linux | Linux Journey (short, friendly lessons) | [linuxjourney.com](https://linuxjourney.com) |
| Docker | Official "Get started" guide | [docs.docker.com](https://docs.docker.com) |
| Networking | *An Introduction to Computer Networks* (open textbook) | [intronetworks.cs.luc.edu](https://intronetworks.cs.luc.edu) |
| Docker networks | How containers talk to each other | [docs.docker.com/engine/network](https://docs.docker.com/engine/network/) |
| Cloud | AWS Cloud Practitioner Essentials | [skillbuilder.aws](https://skillbuilder.aws) |
| What next | Step-by-step learning maps | [roadmap.sh](https://roadmap.sh) |
| People | Learn together with your peers | AWS communities |

## About

**Jacob Kipsang**, Cloud Infrastructure Engineer

- LinkedIn: [linkedin.com/in/jacob-kipsang-3068b1b2](https://www.linkedin.com/in/jacob-kipsang-3068b1b2)
- Email: jkipsang88@gmail.com

**Just start.** One laptop, one hour, one command: `docker run hello-world`.
