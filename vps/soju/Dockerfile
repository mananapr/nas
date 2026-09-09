FROM golang:alpine AS build

RUN apk add --no-cache build-base git
RUN git clone --depth 1 https://git.sr.ht/~emersion/soju /src

WORKDIR /src
RUN go build -o /soju ./cmd/soju \
    && go build -o /sojuctl ./cmd/sojuctl

FROM alpine

RUN addgroup -S soju && adduser -S -G soju soju
RUN mkdir -p /var/lib/soju && chown soju:soju /var/lib/soju
COPY --from=build /soju /usr/local/bin/soju
COPY --from=build /sojuctl /usr/local/bin/sojuctl

USER soju
ENTRYPOINT ["/usr/local/bin/soju"]
