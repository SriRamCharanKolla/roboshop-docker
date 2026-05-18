                ┌────────────────────┐
                │   Host Machine     │
                │ package.json       │
                │ server.js          │
                └─────────┬──────────┘
                          │ COPY
                          ▼

            ┌─────────────────────────┐
            │ Builder Stage           │
            │ node alpine             │
            │ npm install             │
            │ node_modules created    │
            └─────────┬───────────────┘
                      │ COPY --from
                      ▼

            ┌─────────────────────────┐
            │ Production Stage        │
            │ lightweight image       │
            │ non-root user           │
            │ secure container        │
            │ node server.js          │
            └─────────────────────────┘