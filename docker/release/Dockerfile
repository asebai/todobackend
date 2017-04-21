FROM leletir/todobackend-base:latest
MAINTAINER Pierre Leclercq <pleclercq@ingeniance.fr>

# Copy application artifacts
COPY target /wheelhouse

# Install application
RUN . /appenv/bin/activate && \
    pip install --no-index -f /wheelhouse todobackend && \
    rm -rf /wheelhouse