FROM node:18-slim

WORKDIR /usr/src/app
COPY package*.json ./

RUN echo "deb http://mirrors.aliyun.com/debian/ bookworm main non-free contrib" > /etc/apt/sources.list
RUN echo "deb-src http://mirrors.aliyun.com/debian/ bookworm main non-free contrib" >> /etc/apt/sources.list
RUN echo "deb http://mirrors.aliyun.com/debian-security/ bookworm-security main" >> /etc/apt/sources.list
RUN echo "deb-src http://mirrors.aliyun.com/debian-security/ bookworm-security main" >> /etc/apt/sources.list
RUN echo "deb http://mirrors.aliyun.com/debian/ bookworm-updates main non-free contrib" >> /etc/apt/sources.list
RUN echo "deb-src http://mirrors.aliyun.com/debian/ bookworm-updates main non-free contrib" >> /etc/apt/sources.list
RUN echo "deb http://mirrors.aliyun.com/debian/ bookworm-backports main non-free contrib" >> /etc/apt/sources.list
RUN echo "deb-src http://mirrors.aliyun.com/debian/ bookworm-backports main non-free contrib" >> /etc/apt/sources.list
RUN apt-get update

RUN npm config set registry https://registry.npmmirror.com
RUN npm install

COPY . .

# Install Playwright dependencies
RUN npx playwright install --with-deps

RUN npm run build

ARG PORT
ENV PORT=${PORT}

EXPOSE ${PORT}

CMD [ "npm", "start" ]
