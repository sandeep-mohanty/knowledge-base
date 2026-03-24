# 50+ Docker Commands Every Developer & DevOps Engineer Should Know (With Examples & Real-Life Use Cases)

Your complete guide to mastering Docker from basics to production-grade workflows.

Docker has become the backbone of modern application development. Whether you’re building microservices, deploying AI workloads, or setting up CI/CD pipelines, containers make everything consistent, portable, and fast.

This guide covers all essential Docker commands, organized with simple explanations, real-life examples, pro tips, and productivity hacks you can use immediately.

---

## 🧱 What is Docker? (Simple Explanation)

Docker packages applications and their dependencies into portable containers, similar to shipping containers that ensure goods arrive safely anywhere in the world. Containers make deployments predictable, lightweight, fast, and secure — across Dev, QA, Staging, and Production.



---

## 🔰 Basic Docker Commands (Beginner-Friendly)

These are the commands you will use every day.

**1. docker --version** Shows the installed Docker version.
```bash
docker --version
```
📌 **Use case:** Check compatibility before running scripts or Kubernetes.

**2. docker info** Shows system-wide Docker information.
```bash
docker info
```
📌 **Use case:** Debug installation issues.

**3. docker pull** Downloads an image from Docker Hub or a private registry.
```bash
docker pull ubuntu:latest
```

**4. docker images** Lists all available images on the system.
```bash
docker images
```

**5. docker run** Creates and starts a new container from an image.
```bash
docker run -it ubuntu bash
```
📌 **Use case:** Open a shell in a fresh environment.

**6. docker ps** Shows running containers.
```bash
docker ps
```

**7. docker ps -a** Shows all containers (running + stopped).
```bash
docker ps -a
```

**8. docker stop** Stops a running container.
```bash
docker stop web_app
```

**9. docker start** Starts a stopped container.
```bash
docker start web_app
```

**10. docker rm** Removes a container.
```bash
docker rm old_container
```

**11. docker rmi** Removes an image.
```bash
docker rmi my_image:v1
```

**12. docker exec** Run a command inside a running container.
```bash
docker exec -it app bash
```
💡 **HACK:** Use this to debug production without SSH.

---

## 🚀 Intermediate Docker Commands (Most Used in DevOps)



**13. docker build** Build an image from a Dockerfile.
```bash
docker build -t my_app:latest .
```

**14. docker commit** Save changes made inside a container as a new image.
```bash
docker commit test_container my_image:v1
```

**15. docker logs** View container logs.
```bash
docker logs payment_service
```

**16. docker inspect** Detailed information about a container or image.
```bash
docker inspect mysql_container
```

**17. docker stats** Live CPU, memory & network usage.
```bash
docker stats
```

**18. docker cp** Copy files between container ↔ host.
```bash
docker cp app:/var/log/app.log .
```

**19. docker rename** Rename a container.
```bash
docker rename old_name new_name
```

**20. docker network ls** List all Docker networks.
```bash
docker network ls
```

**21. docker network create** Create a network.
```bash
docker network create backend
```

**22. docker network inspect** Show details of a network.
```bash
docker network inspect backend
```

**23. docker network connect** Connect a container to a network.
```bash
docker network connect backend app1
```

**24. docker volume ls** List volumes.
```bash
docker volume ls
```

**25. docker volume create** Create a volume.
```bash
docker volume create dbdata
```

**26. docker volume inspect** Inspect volume details.
```bash
docker volume inspect dbdata
```

**27. docker volume rm** Delete a volume.
```bash
docker volume rm oldvol
```

---

## 🧰 Advanced Docker Commands (CI/CD + Production Use)



**28. docker-compose up** Start multi-container apps.
```bash
docker-compose up -d
```
📌 **Use case:** Spin up a full microservices environment.

**29. docker-compose down** Stop and remove all services.
```bash
docker-compose down
```

**30. docker-compose logs** View logs of all services.
```bash
docker-compose logs -f
```

**31. docker-compose exec** Run a command inside a compose-managed container.
```bash
docker-compose exec db bash
```

**32. docker save** Save an image as a .tar file.
```bash
docker save -o backup.tar myimage
```

**33. docker load** Restore image from .tar.
```bash
docker load < backup.tar
```

**34. docker export** Export container filesystem.
```bash
docker export app > app_fs.tar
```

**35. docker import** Import a filesystem as an image.
```bash
docker import app_fs.tar new_image
```

**36. docker system df** Check Docker disk usage.
```bash
docker system df
```

**37. docker system prune** Clean unused Docker resources.
```bash
docker system prune -a
```
⚠ **WARNING:** Removes old images too!

**38. docker tag** Tag an image.
```bash
docker tag myapp:latest repo/myapp:v1
```

**39. docker push** Push image to registry.
```bash
docker push repo/myapp:v1
```

**40. docker login** Authenticate with registry.

**41. docker logout** Logout of registry.

---

## 🐝 Docker Swarm (Orchestration)



**42. docker swarm init** Start a Swarm cluster.
```bash
docker swarm init
```

**43. docker service create** Create a service.
```bash
docker service create --name web nginx
```

**44. docker stack deploy** Deploy a full stack.
```bash
docker stack deploy -c docker-compose.yml mystack
```

**45. docker stack rm** Remove a stack.
```bash
docker stack rm mystack
```

---

## 🎯 Checkpoint Commands (Advanced Debugging)

**46. Checkpoint Create / List / Remove** Used to freeze and restore containers (experimental).
```bash
docker checkpoint create app cp1
docker checkpoint ls app
docker checkpoint rm app cp1
```

---

## ⚡ 25 Real-Life Useful Docker Commands & Hacks

Here are additional real-world commands that engineers use daily:

* **✔ Show environment variables** `docker exec app env`
* **✔ Tail logs in real time** `docker logs -f app`
* **✔ Remove ALL stopped containers** `docker container prune`
* **✔ Remove dangling images** `docker image prune`
* **✔ Restart a container** `docker restart app`
* **✔ Show container IP** `docker inspect -f '{{.NetworkSettings.IPAddress}}' app`
* **✔ Port mapping (host → container)** `docker run -p 8080:80 nginx`
* **✔ Limit memory usage** `docker run -m 512m nginx`
* **✔ Limit CPU usage** `docker run --cpus="1.5" nginx`
* **✔ List processes inside a container** `docker top app`
* **✔ View port mappings** `docker port app`
* **✔ Remove ALL unused data (dangerous!)** `docker system prune -a --volumes`

---

## 💡 Pro Tips & Best Practices for Docker Pros

**1. ⭐ Use .dockerignore to speed up builds** Exclude unnecessary files (`node_modules`, `logs`, `configs`).

**2. ⭐ Prefer lightweight base images** Switch from Ubuntu → Alpine to reduce image size by **90%**.

**3. ⭐ Use multi-stage Docker builds** Build → optimize → ship minimal final image.

**4. ⭐ Never store secrets in images** Use environment variables or vaults.

**5. ⭐ Don’t run containers as root** Use:  
`USER appuser`

---

The modern developer's toolkit is incomplete without Docker. By mastering these commands, you're not just running containers; you're orchestrating efficient, scalable environments.