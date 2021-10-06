# Custom definition

## Description
Now we know how to store and retrieve data from basicProfile which is a definition already provided by IDX. But what if we want to store data that is specific to our application. For this reason let’s assume that we are building application that store favourite quote of a user. How can we do it?
First we will need to define our custom JSON schema that looks like

```json
{
  "$schema": "http://json-schema.org/draft-07/schema",
  "title": "FigmentLearn",
  "description": "Web3 learning with Figment Learn",
  "type": "object",
  "properties": {
    "text": {
      "type": "string",
      "maxLength": 300
    },
    "author": {
      "type": "string",
      "maxLength": 150
    }
  }
}
```

Here you can see that we have only 2 properties for our schema: text and author which are both strings.

Once we have this schema we have to use IDX CLI to first generate new DID and set node url. In order to do that run:

```
```

Once we have this done, we can publish our schema and get SchemaURL because that is what we need in order deploy our definition.

```bash
idx schema:publish did:key:z6MkvidREB5ASdWEihxhRatHBFEWzhM8tbk4Bzn1iEWUT1vn ‘{ "$schema": "http://json-schema.org/draft-07/schema", "title": "FigmentLearn", "description": "Web3 learning with Figment Learn", "type": "object", "properties": { "text": { "type": "string", "maxLength": 300 }, "author": { "type": "string", "maxLength": 150 } } }’
```

This should return the url which looks similar to `ceramic://k3y52l7qbv1fryo5num99f2umi34cae2yn13wyo4hs3o0d4jai1hq1owg1ep04w74`.  We can take that URL and use it  create definition:

```bash
idx definition:create did:key:z6MkvidREB5ASdWEihxhRatHBFEWzhM8tbk4Bzn1iEWUT1vn --schema=' ceramic://k3y52l7qbv1fryo5num99f2umi34cae2yn13wyo4hs3o0d4jai1hq1owg1ep04w74' --name=“Bartosz test” --description=“This “is a test"
```

Here we provide name and description to our definition. This will return StreamID for our definition that looks something similar to: `kjzl6cwe1jw146qq6gh9j04b43jycgypjs8v5gqhtnurtym0q1nma0vr3vd6px3`

Now we need to create an alias for that definition’s StreamID.

```js
export const aliases = {
  figment: 'kjzl6cwe1jw146qq6gh9j04b43jycgypjs8v5gqhtnurtym0q1nma0vr3vd6px3',
};
```

Now once this is done we can go ahead and try to store our favourite quote using IDX.

```js
const saveQuote = async (values: QuoteSchemaT) => {
  setSaving(true);
  const {text, author} = values;

  try {
    await idx.set(IdxSchema.Figment, {text, author});
    setCustomSchemaSaved(true);
  } catch (error) {
    setError(error);
  } finally {
    setSaving(false);
  }
};
```

Here we use familiar set method but instead of passing `basicProfile` as first parameter we pass alias to our custom definition `figment` and as a second parameter we pass ing data defined in our custom json schema.

Now we can test if the data indeed got stored to Ceramic.

```js
const handleGetQuote = async () => {
  try {
    const resp = await idx.get<QuoteSchemaT>(IdxSchema.Figment);
    setCustomSchemaData(resp);
  } catch (error) {
    setError(error);
  } finally {
    setFetching(false);
  }
};
```

Here we use get method and we provide our custom alias.
