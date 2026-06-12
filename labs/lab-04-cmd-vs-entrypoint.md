# Lab 04 — CMD vs ENTRYPOINT

## What This Lab Covers

This lab covers two Dockerfile instructions that control what happens when a container starts: CMD and ENTRYPOINT. At first glance they seem interchangeable. They are not. Understanding the difference is what lets you build images that are both predictable and flexible, and it is what helps you debug containers that are not starting the way you expect.

This lab covers the CMD instruction, the ENTRYPOINT instruction, how they behave differently at runtime, how to use them together, and how to override ENTRYPOINT from the command line.

---

## Before the Commands: What You Need to Know

**The problem this section solves**

Say you want to build an image that always runs a specific program when it starts, but you also want to be able to pass different arguments to that program without rewriting the Dockerfile every time. That is the exact scenario CMD and ENTRYPOINT are designed for.

**What CMD does**

CMD defines the default command that runs when a container starts. If you pass anything after the image name when running the container, it completely replaces whatever CMD specifies. The whole command gets swapped out.

    docker run ubuntu sleep 10

Here sleep 10 replaces whatever CMD was set to in the Ubuntu image entirely.

**What ENTRYPOINT does**

ENTRYPOINT also defines what runs when a container starts, but it behaves differently at runtime. Whatever you pass after the image name gets appended to the ENTRYPOINT rather than replacing it. The executable stays fixed.

The limitation of ENTRYPOINT alone is that if you forget to pass an argument, the command runs incomplete and throws an error.

**Why you use them together**

Using CMD and ENTRYPOINT together solves both problems. ENTRYPOINT locks in the executable that always runs. CMD sets a default argument that gets used if you do not pass one at runtime. If you do pass an argument, it overrides CMD but leaves ENTRYPOINT untouched.

This gives you a fixed executable with a sensible default and the flexibility to override the argument whenever you need to. That is the combination worth remembering.

**JSON array format**

When using CMD and ENTRYPOINT together, both must be written in JSON array format. Shell form does not work reliably in this case. Each part of the command is a separate element in the array.

    ENTRYPOINT ["sleep"]
    CMD ["5"]

The executable and its arguments are always separate elements. Never combine them into a single string.

---

## Dockerfile Instructions

**CMD**

CMD sets the default command that runs when the container starts. JSON array format is preferred, especially when combining with ENTRYPOINT.

    CMD ["sleep", "5"]

If you pass a command after the image name at runtime, it replaces CMD entirely.

    docker run ubuntu-sleeper sleep 10

The command at startup becomes sleep 10. The original CMD is gone.

**ENTRYPOINT**

ENTRYPOINT sets the fixed executable that always runs when the container starts. Whatever you pass at runtime gets appended to it rather than replacing it.

    ENTRYPOINT ["sleep"]

Running this image with a number appended:

    docker run ubuntu-sleeper 10

The command at startup becomes sleep 10. You are supplying the argument, not the command.

**CMD and ENTRYPOINT together**

Used together, ENTRYPOINT holds the executable and CMD holds the default argument. If nothing is passed at runtime, the default runs. If something is passed, it overrides CMD only.

    ENTRYPOINT ["sleep"]
    CMD ["5"]

Running the image with no arguments starts with sleep 5. Running it with 10 appended starts with sleep 10.

**Overriding ENTRYPOINT at runtime**

If you need to change the ENTRYPOINT itself at runtime, use the --entrypoint flag in the docker run command.

    docker run --entrypoint sleep2.0 ubuntu-sleeper 10

The command at startup becomes sleep2.0 10. This is useful when you need to swap out the executable entirely without rebuilding the image.

---

## AWS Connection

**ECS task definitions.** In ECS, task definitions have two fields that map directly to what you learned here. The entryPoint field overrides ENTRYPOINT and the command field overrides CMD. When a container in ECS is not starting correctly, one of the first things to check is whether those fields are set correctly in the task definition and whether they match what the Dockerfile expects.

**Lambda container images.** When deploying a Lambda function as a container image, CMD is used to specify the handler (the function that Lambda invokes). Understanding that CMD is overridable at runtime is what makes Lambda container images flexible across different deployment configurations.

---

## Troubleshooting

*No troubleshooting entries for this lab. The six question lab completed without errors worth documenting.*

---

## Key Observations

- CMD and ENTRYPOINT look similar on the surface but behave completely differently at runtime. CMD gets replaced entirely when you pass anything after the image name. ENTRYPOINT stays fixed and receives what you pass as an argument. That single distinction is the whole lesson.

- The Ubuntu container exiting immediately, which first came up in Lab 01, connects directly to this. The Ubuntu image has bash set as its CMD. Docker does not attach a terminal by default, bash finds nothing to connect to and exits. Now that CMD and ENTRYPOINT make sense, that behavior makes more sense too.

- Using CMD and ENTRYPOINT together is the practical default. ENTRYPOINT for what always runs, CMD for the default argument that can be swapped out. The JSON array format is not optional when combining them. It is required for them to work correctly together.

- Knowing that --entrypoint exists matters less for day to day work and more for debugging. When a container is not behaving the way you expect, being able to override the entrypoint from the command line and test different executables without rebuilding the image is a useful tool to have.
