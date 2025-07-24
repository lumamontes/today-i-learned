### graphql request queries and mutations


Today i learned how to use graphql request to make queries and mutations to my backend.

First, i created a <code>executeGraphql</code> method that i can reuse across my application frontend:

```
export async function executeGraphQL<T = any>(
  query: string,
  variables?: Record<string, any>
): Promise<T> {
  try {
    const result = await graphqlClient.request<T>(query, variables);
    return result;
  } catch (error) {
    throw error;
  }

```

Then, based on my backend graphql configuration, i create the query and mutation variables:

```
const GET_COMPANIES_QUERY = `
  query GetCompanies {
    companies {
      _id
      name
      cnpj
      createdAt
      updatedAt
    }
  }
`;

const CREATE_COMPANY_MUTATION = `
  mutation CreateCompany($input: CreateCompanyInput!) {
    createCompany(input: $input) {
        name
        cnpj
        status
    }
  }
`;

```

After that, i can just use the consts with the query and the mutation and pass it to my graphql client request.

```
const data = await executeGraphQL<{ companies: Company[] }>(
    GET_COMPANIES_QUERY
);

const data = await executeGraphQL<{ createCompany: CreateCompanyInput }>(
CREATE_COMPANY_MUTATION,
{ input: company }
);

```