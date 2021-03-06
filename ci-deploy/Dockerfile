# This Dockerfile is just used to run on Travis CI in an environment that can easily and repeatedly
# install our build dependencies.
FROM debian:stable

RUN apt-get update && \
    apt-get install -y ca-certificates curl rsync git unzip \
    default-jre                                             \
    libfontconfig1 libgomp1 libxml2                         \
    python3 python3-pip                                     \
    fonts-dejavu fonts-droid-fallback fonts-liberation fonts-symbola fonts-unfonts-core

# Dependency lines above are:
# - General
# - validator
# - Prince
# - Highlighter
# - fonts

RUN pip3 install bs-highlighter

# Dependecies of prince_11.3-1_debian8.0_amd64.deb (not used) are libc6 libcurl3 libfontconfig1
# libfreetype6 libgif4 libgomp1 libjpeg62-turbo libpng12-0 libssl1.0.0 libtiff5 libxml2 zlib1g.
# Above we install only the subset that's needed to make Prince work.

ADD https://www.princexml.com/download/prince-11.3-linux-generic-x86_64.tar.gz /whatwg/prince.tar.gz

RUN cd /whatwg && \
    tar xzf prince.tar.gz && \
    echo /whatwg/prince | /whatwg/prince-11.3-linux-generic-x86_64/install.sh && \
    echo '@font-face { font-family: serif; src: local("Symbola") }' >> /whatwg/prince/lib/prince/style/fonts.css && \
    rm -rf /whatwg/prince.tar.gz /whatwg/prince-11.3-linux-generic-x86_64
ENV PATH="/whatwg/prince/bin:${PATH}"

ADD pdfsizeopt/pdfsizeopt.single /whatwg/pdfsizeopt/bin/pdfsizeopt
ADD https://github.com/pts/pdfsizeopt/releases/download/2017-01-24/pdfsizeopt_libexec_linux-v3.tar.gz /whatwg/pdfsizeopt_libexec.tar.gz
RUN cd /whatwg && \
    tar xzf pdfsizeopt_libexec.tar.gz && \
    mv pdfsizeopt_libexec/* pdfsizeopt/bin/ && \
    rm -rf pdfsizeopt_libexec.tar.gz pdfsizeopt_libexec
ENV PATH="/whatwg/pdfsizeopt/bin:${PATH}"

COPY --from=whatwg/wattsi:latest /whatwg/wattsi/bin/wattsi /bin/wattsi

ADD html-build /whatwg/html-build

# Note: we do not ADD /whatwg/html, but instead mount it using --volume in .travis.yml, since it
# contains the deploy_key, and thus should not be part of the image. The image is cached, publicly,
# on Docker Hub.
ENV HTML_SOURCE /whatwg/html

ARG travis_pull_request
ARG is_test_of_html_build_itself
ENV TRAVIS_PULL_REQUEST=${travis_pull_request}
ENV IS_TEST_OF_HTML_BUILD_ITSELF=${is_test_of_html_build_itself}

ENV SKIP_BUILD_UPDATE_CHECK=true
ENTRYPOINT ["bash", "/whatwg/html-build/ci-deploy/inside-container.sh"]
