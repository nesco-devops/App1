FROM node:22-alpine AS build

WORKDIR /app

ARG NEXT_PUBLIC_API_URL=/api
ENV NEXT_PUBLIC_API_URL=${NEXT_PUBLIC_API_URL}
ENV NEXT_TELEMETRY_DISABLED=1

COPY package.json package-lock.json ./
RUN npm ci

COPY . .

RUN npm run build


FROM node:22-alpine AS runtime

RUN apk upgrade --no-cache

WORKDIR /app

ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1
ENV PORT=3000
ENV HOSTNAME=0.0.0.0

RUN rm -rf /usr/local/lib/node_modules/npm \
    /opt/yarn-* \
    /usr/local/bin/npm \
    /usr/local/bin/npx \
    /usr/local/bin/yarn \
    /usr/local/bin/yarnpkg
    
RUN apk del --no-cache apk-tools libapk zlib

COPY --from=build --chown=node:node /app/.next/standalone ./
COPY --from=build --chown=node:node /app/.next/static ./.next/static
COPY --from=build --chown=node:node /app/public ./public

USER node

# Tester Sharp après le retrait des paquets système.
RUN node -e "require('sharp')({create:{width:1,height:1,channels:3,background:{r:0,g:0,b:0}}}).png().toBuffer().then(() => console.log('Sharp PNG OK')).catch(error => { console.error(error); process.exit(1); })"

EXPOSE 3000

CMD ["node", "server.js"]
