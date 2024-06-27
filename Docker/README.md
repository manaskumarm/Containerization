# Docker
A container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another. A Docker container image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings.

![image](https://github.com/manaskumarm/Containerization/assets/14363425/c6719aa8-1ced-48f9-8704-dc5e83087603)


![image](https://github.com/manaskumarm/Containerization/assets/14363425/cfc12847-c11f-4379-adbb-c0444c221c25)

## Multistage
A Docker multi-stage build allows you to use multiple _FROM_ statements in your Dockerfile. Each _FROM_ statement represents a distinct stage in the build process. This technique helps optimize Docker images for size, security, and build speed.

**Multiple Stages**:
Each FROM instruction starts a new stage, creating a separate image.
You can name each stage using the AS keyword for easier reference.

**Selective Copying**:
Use the COPY --from instruction to selectively copy files and directories between stages.
This approach brings in only the necessary artifacts from previous stages, reducing the final image size.

```
# Stage 1: Build the application
FROM node:16 AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Create the production image
FROM node:16-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist
EXPOSE 8082
CMD ["node", "dist/index.js"]
```
### Advantages
**Smaller Image Size**: When you use different stages for building and running an application, you can exclude unnecessary dependencies and intermediate files from the final image. This results in a smaller image, which makes it faster to download and deploy.

**Improved Security**: Multi-stage builds allow you to minimize the attack surface by including only the essential components in the final image. By doing so, you reduce the risk of vulnerabilities and exploits.

**Faster Builds**: Organizing the build process into stages enables you to leverage Docker’s layer caching mechanism more efficiently. If you make changes to your code, only the affected stages need to be rebuilt, saving time during development.
