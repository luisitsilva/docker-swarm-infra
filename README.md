## Prerequisites

Docker Engine 24+ with Swarm mode initialized
A Caddy instance (or service) already running outside this stack, attached to caddy-public

Initialize Swarm if you haven't:
docker swarm init

## Deploy
docker stack deploy -c docker-compose.yaml <stack_name>

Check status:
docker stack services devops
docker stack ps devops

Remove the stack:
docker stack rm devops

## Docker networks
This stack declares caddy-public as external. 

## Create the network (one time, before deploying)
docker network create \
  --driver overlay \
  --attachable \
  --scope swarm \
  caddy-public

--driver overlay → swarm-scoped

--attachable → allows standalone (non-service) containers, like a Caddy proxy running via docker run, to join

## Verify
docker network ls | grep caddy-public
# <id>   caddy-public   overlay   swarm

## Idempotent bootstrap
For scripts / Makefiles, create only if missing or wrong scope:
docker network inspect caddy-public --format '{{.Scope}}' 2>/dev/null | grep -q swarm \
  || docker network create --driver overlay --attachable caddy-public


## Notes
Only one stack should own the caddy-public network definition. Downstream stacks reference it with external: true.

Caddy should be attached to caddy-public so it can reach services published on that network by their service name.

--detach=false is not required; tasks are created in the background by default.

## License
MIT
