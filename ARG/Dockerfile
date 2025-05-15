ARG version
#FROM almalinux:${version}
FROM almalinux:${version:-8}
ARG COURSE=Docker
ARG role=devops
ARG Duration=120HRS
ENV Duration=${Duration}
RUN echo "Course is: ${COURSE}, role is: ${role}, Duration is: ${Duration}"
CMD [ "sleep","100" ]