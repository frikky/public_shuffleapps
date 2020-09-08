# Base our app image off of the WALKOFF App SDK image
FROM frikky/shuffle:app_sdk as base

# We're going to stage away all of the bloat from the build tools so lets create a builder stage
FROM base as builder

# Install all alpine build tools needed for our pip installs
RUN apk --no-cache add --update alpine-sdk libffi libffi-dev musl-dev openssl-dev 

# Install all of our pip packages in a single directory that we can copy to our base image later
RUN mkdir /install
WORKDIR /install
COPY requirements.txt /requirements.txt
RUN pip install --prefix="/install" -r /requirements.txt




# Switch back to our base image and copy in all of our built packages and source code
FROM node:12-slim 

RUN apt-get update \
  && apt-get install -y wget gnupg \
	&& wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
	&& sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list' \
	&& apt-get update \
	&& apt-get install -y google-chrome-stable fonts-ipafont-gothic fonts-wqy-zenhei fonts-thai-tlwg fonts-kacst fonts-freefont-ttf libxss1 \
	--no-install-recommends \
	&& rm -rf /var/lib/apt/lists/*

RUN npm i puppeteer \
# Add user so we don't need --no-sandbox.
# same layer as npm install to keep re-chowned files from using up several hundred MBs more space
	&& groupadd -r pptruser && useradd -r -g pptruser -G audio,video pptruser \
	&& mkdir -p /home/pptruser/Downloads \
	&& chown -R pptruser:pptruser /home/pptruser \
	&& chown -R pptruser:pptruser /node_modules

RUN apt-get update
RUN apt-get -y install python3

COPY --from=builder /install /usr/local
COPY src /app

# Run everything after as non-privileged user.
#USER pptruser

# Finally, lets run our app!
WORKDIR /app
RUN ls /app
CMD python3 app.py --log-level DEBUG
