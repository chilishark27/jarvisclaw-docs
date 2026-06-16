# Smart Router

Set the model field to a router alias instead of a specific model ID. Smart Router analyses your request and selects the optimal upstream model automatically — no code changes needed when new models are added.

## auto

Smart Router picks the best available model for each request based on capability, latency, and cost.

```python
response = client.chat.completions.create(
    model="auto",
    messages=[{"role": "user", "content": "Hello!"}]
)
```

## free

Routes exclusively to free-tier models. Zero token cost — great for development and experimentation.

```python
response = client.chat.completions.create(
    model="free",
    messages=[{"role": "user", "content": "Hello!"}]
)
```

## eco

Selects the most cost-efficient paid model that can handle the task. Minimises spend per token.

```python
response = client.chat.completions.create(
    model="eco",
    messages=[{"role": "user", "content": "Hello!"}]
)
```

## premium

Routes to the highest-capability model available. Best for complex reasoning and production workloads.

```python
response = client.chat.completions.create(
    model="premium",
    messages=[{"role": "user", "content": "Hello!"}]
)
```
