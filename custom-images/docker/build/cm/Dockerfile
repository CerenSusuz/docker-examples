# escape=`

# Initialize build stages

ARG BASE_IMAGE
ARG SXA_IMAGE
ARG SPE_IMAGE
ARG TOOLING_IMAGE
ARG SOLUTION_IMAGE
ARG HORIZON_RESOURCES_IMAGE

FROM ${SOLUTION_IMAGE} as solution
FROM ${TOOLING_IMAGE} as tooling
FROM ${SPE_IMAGE} as spe
FROM ${SXA_IMAGE} as sxa
FROM ${HORIZON_RESOURCES_IMAGE} as horizon_resources
FROM ${BASE_IMAGE}


SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# Add development tools
# Copy development tools and entrypoint. Development tools are copied in from the tooling image to C:\tools. Along with providing an ENTRYPOINT for local development, these will be used later on to apply transforms.
COPY --from=tooling \tools\ \tools\


# Set working directory
# Since this uses an IIS image, most of our customizations will be done within C:\inetpub\wwwroot, so this is set to the working directory.
WORKDIR C:\inetpub\wwwroot


# Add Sitecore modules
# Sitecore modules are added according to the required instructions for the cm role. This example includes Sitecore PowerShell Extensions (SPE) and Sitecore Experience Accelerator (SXA). 

# Add SPE module
COPY --from=spe \module\cm\content .\

# Add SXA module
COPY --from=sxa \module\cm\content .\
COPY --from=sxa \module\tools \module\tools
RUN C:\module\tools\Initialize-Content.ps1 -TargetPath .\; `
    Remove-Item -Path C:\module -Recurse -Force;

# Add horizon module
COPY --from=horizon_resources \module\cm\content \inetpub\wwwroot

# Add files

# Files from the solution build image are copied in. The output files are stored on the example solution image at \artifacts\website.
# Copy solution website files
COPY --from=solution \artifacts\website\ .\


# Add transforms
# The cm service has an example of both solution and role transform files. Solution transforms are copied in first ( the output transforms are stored on the example solution image at \artifacts\transforms\), and then role transforms.

# Copy solution transforms
COPY --from=solution \artifacts\transforms\ \transforms\solution\

# Copy role transforms
COPY .\transforms\ \transforms\role\

# Apply transforms
#  the solution and then role transforms are applied to the web root.
# Perform solution transforms
RUN C:\tools\scripts\Invoke-XdtTransform.ps1 -Path .\ -XdtPath C:\transforms\solution\DockerExamples.Website

# Perform role transforms
RUN C:\tools\scripts\Invoke-XdtTransform.ps1 -Path .\ -XdtPath C:\transforms\role