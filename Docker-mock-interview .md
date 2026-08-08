1. What are CMD and ENTRYPOINT? What is the difference?

1) Both CMD and ENTRYPOINT define what runs when a Docker container starts, but they have different purposes. ENTRYPOINT defines the main executable of the container, while CMD provides default arguments or a default command.

2) The important difference is how they behave when we provide a command while running the container. CMD can be completely overridden, whereas ENTRYPOINT normally remains the executable and the supplied command becomes its arguments.

3) For production images, I generally use ENTRYPOINT when the container represents a specific executable or application, and CMD for default arguments that users may want to override.

I prefer the exec form, such as ENTRYPOINT ["python", "app.py"], rather than shell form, because it provides better signal handling and process behavior.”

ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8080"]

ENTRYPOINT defines what the container is; CMD defines the default command or arguments.”

===================================================================
