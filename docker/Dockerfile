FROM bitwalker/alpine-elixir-phoenix:1.7.1

RUN apk --no-cache --update add automake libtool inotify-tools autoconf python

EXPOSE 4000

ENV MIX_ENV='prod'
ENV COIN='ETH'
ENV ETHEREUM_JSONRPC_VARIANT='ganache'
ENV ETHEREUM_JSONRPC_HTTP_URL='http://host.docker.internal:8545'
ENV ETHEREUM_JSONRPC_WS_URL='ws://host.docker.internal:8545'

ENV DATABASE_URL='postgresql://postgres:postgres@host.docker.internal:5432/explorer?ssl=false'
ENV PORT=4000
ENV SECRET_KEY_BASE="RMgI4C1HSkxsEjdhtGMfwAHfyT6CKWXOgzCboJflfSm4jeAlic52io05KB6mqzc5"

# Cache elixir deps
ADD mix.exs mix.lock ./
ADD apps/block_scout_web/mix.exs ./apps/block_scout_web/
ADD apps/explorer/mix.exs ./apps/explorer/
ADD apps/ethereum_jsonrpc/mix.exs ./apps/ethereum_jsonrpc/
ADD apps/indexer/mix.exs ./apps/indexer/
ADD _build ./_build

RUN mix deps.get

ADD . .

ARG COIN
RUN if [ "$COIN" != "" ]; then sed -i s/"POA"/"${COIN}"/g apps/block_scout_web/priv/gettext/en/LC_MESSAGES/default.po; fi

# Run forderground build and phoenix digest
RUN mix compile

# Add blockscout npm deps
RUN cd apps/block_scout_web/assets/ && \
    npm install && \
    npm run deploy && \
    cd -

RUN cd apps/explorer/ && \
    npm install && \
    cd -

# RUN mix do ecto.drop --force, ecto.create, ecto.migrate

# USER default

CMD sleep 3 && \
  mix do ecto.drop --force --no-compile, ecto.create --no-compile --quiet, ecto.migrate --no-compile --quiet && \
  mix phx.server
