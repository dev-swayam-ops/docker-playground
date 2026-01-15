# Module 10: Exercises, Solutions, Cheatsheet & Quiz

## Exercises

1. **Set memory limits**: Run a container with 256MB memory limit using `--memory=256m`.
2. **Set CPU limits**: Run a container restricted to 0.5 CPUs using `--cpus=0.5`.
3. **Monitor container stats**: Use `docker stats` to track resource usage over time.
4. **Compare image sizes**: Build two versions of an image (standard and slim base) and compare sizes.
5. **Optimize Dockerfile**: Reduce a Dockerfile by combining RUN commands with && and ; operators.
6. **Create multi-stage build**: Build an image using two stages to reduce final size.
7. **Verify layer caching**: Make multiple changes to a Dockerfile and observe which layers are cached.
8. **Use .dockerignore**: Create a .dockerignore file to exclude unnecessary files from the build.
9. **Measure startup time**: Time container startup with and without resource limits.
10. **Profile image layers**: Use `docker history [image]` to analyze layer sizes and content.