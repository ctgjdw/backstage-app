FROM backstage-base

COPY --chown=node:node app-config.production.yaml ./
ENV NODE_ENV production
CMD ["node", "packages/backend", "--config", "app-config.production.yaml"]