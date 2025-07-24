# tanstack router data loading

TIL i learned that we can data loading in tanstack router in a very simple way.

Given this route:

```
export const Route = createFileRoute('/_authenticated/companies/')({
  component: RouteComponent,
})

```

We can then add the property loader and make the data fetching request:

```
export const Route = createFileRoute('/_authenticated/companies/')({
  component: RouteComponent,
  loader: async () => {
    const companies = await companiesApiService.getCompanies()
    return { companies }
  }
})

```

Then, in our RouteComponent, we can access the companies property with:

```
function RouteComponent() {
  const { companies } = useLoaderData({ from: Route.id })
  /// rest of code
}
```