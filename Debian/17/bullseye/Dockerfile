# vim:set ft=dockerfile:
#
# Copyright The CloudNativePG Contributors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
# 
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

FROM curlimages/curl as downloader

RUN curl --silent \
         --fail \
         --location \
         --output artifact.zip \
         "https://github.com/timescale/pgvectorscale/releases/download/0.6.0/pgvectorscale-0.6.0-pg$(echo 17.4-bullseye | awk -F. '{print $1}')-amd64.zip" \
				&& unzip artifact.zip && \
				rm artifact.zip && \
				curl --silent \
         --fail \
         --location \
				 --output pg_jsonschema-amd64-linux-gnu.deb \
        "https://github.com/supabase/pg_jsonschema/releases/download/v0.3.3/pg_jsonschema-v0.3.3-pg$(echo 17.4-bullseye | awk -F. '{print $1}')-amd64-linux-gnu.deb"





FROM postgres:17.4-bullseye

# Do not split the description, otherwise we will see a blank space in the labels
LABEL name="PostgreSQL Container Images" \
      vendor="The CloudNativePG Contributors" \
      version="${PG_VERSION}" \
      release="21" \
      summary="PostgreSQL Container images." \
      description="This Docker image contains PostgreSQL and Barman Cloud based on Postgres 17.4-bullseye."

LABEL org.opencontainers.image.description="This Docker image contains PostgreSQL and Barman Cloud based on Postgres 17.4-bullseye."

COPY requirements.txt /
COPY --from=downloader /home/curl_user .


# Install additional extensions
RUN set -xe; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		"postgresql-${PG_MAJOR}-pgaudit" \
		"postgresql-${PG_MAJOR}-pgvector" \
		"postgresql-${PG_MAJOR}-pg-failover-slots" \
    "postgresql-${PG_MAJOR}-repack" \
    "postgresql-${PG_MAJOR}-cron" \
    "postgresql-${PG_MAJOR}-partman" \
    "postgresql-${PG_MAJOR}-pgtap" \
    "postgresql-${PG_MAJOR}-semver" \
    "postgresql-${PG_MAJOR}-powa" \
    "postgresql-${PG_MAJOR}-pg-qualstats" \
    "postgresql-${PG_MAJOR}-pg-stat-kcache" \
    "postgresql-${PG_MAJOR}-pg-wait-sampling" \
    "postgresql-${PG_MAJOR}-pg-track-settings" \
    "postgresql-${PG_MAJOR}-hypopg" \
    "pgbackrest" \
	; \
  dpkg --install --force-depends --force-not-root --force-overwrite pgvectorscale*amd64.deb; \
  dpkg --install --force-depends --force-not-root --force-overwrite pg_jsonschema*amd64-linux-gnu.deb; \
	rm -fr /tmp/* ; \
	rm -rf /var/lib/apt/lists/*;

# Install barman-cloud
RUN set -xe; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		python3-pip \
		python3-psycopg2 \
		python3-setuptools \
	; \
	pip3 install  --upgrade pip; \
	# TODO: Remove --no-deps once https://github.com/pypa/pip/issues/9644 is solved
	pip3 install  --no-deps -r requirements.txt; \
	rm -rf /var/lib/apt/lists/*;

# Change the uid of postgres to 26
RUN usermod -u 26 postgres
USER 26
