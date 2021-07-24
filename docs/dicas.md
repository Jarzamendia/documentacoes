# Dicas para o uso do Mkdocs

## Code Sampling

```typescript
const serviceEntityPage = (
  <EntityLayout>
    <EntityLayout.Route path="/" title="Overview">
      <Grid container spacing={3} alignItems="stretch">
        <Grid item md={6}>
          <EntityAboutCard variant="gridItem" />
        </Grid>
      </Grid>
    </EntityLayout.Route>
    <EntityLayout.Route path="/docs" title="Docs">
      <EntityTechdocsContent />
    </EntityLayout.Route>
  </EntityLayout>
);
```

```python
def getUsersInGroup(targetGroup, secure=False):

    if __debug__:
        print('targetGroup=[' + targetGroup + ']')
```