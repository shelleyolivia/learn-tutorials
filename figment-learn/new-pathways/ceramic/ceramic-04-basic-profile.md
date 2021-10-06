# Basic profile

## Description
Now that we have our DID and we know how to authenticate using 3ID Connect, we can look into how to associate data to our DID. But first let’s talk about what is happening under the hood of IDX.


[image:D00B35D9-C5F7-4C41-9C00-E6429BD8FF27-498-00008D85E242EF45/idx-simple.png]

Every DID has an index which is a map of DefinitiionID and RecordID. Record defined by RecordID is the data that you want to store. But how IDX knows about the format of the data? That’s where Schema comes in. Schema is a JSON schema that describes the format of the data. So if you want to submit a Record for that schema you need to conform to that schema. Definition is a house of schema StreamID also with the name and description. So if you want to store something to IDX that is associated to your DID you need to know DefinitionID which tells IDX what data will come in and you need to provider Record with an actually data. If the data that you are trying to submit does not conform to schema, your request will fail.
Since DefinitionID is just a StreamID which is not really human readable, IDX has a concept of aliases which are just human readable names for Definitions identified by DefinitionIDs. One of the default definition provided by IDX is called BasicProfile. It is definition used to hold basic profile information like name, description, avatar etc.
In this tutorial we will learn how you can write data to BasicProfile and then read it.


## Steps
Make sure that you are authenticated first.

```js
const saveBasicProfile = async (values: BasicProfile) => {
  setSaving(true);
  const {name} = values;

  try {
    await idx.set(IdxSchema.BasicProfile, {name});
    setBasicProfileSaved(true);
  } catch (error) {
    setError(error);
  } finally {
    setSaving(false);
  }
};
```

Here we use idx instance and we call set on it providing alias name `basicProfile` and data that we want to save. In this case we only store name to Ceramic.



Now that we have our data stored with IDX we can see if we can retrieve it.

```js
const handleGetName = async () => {
  try {
    setFetching(true);
    const resp = await idx.get<BasicProfile>(IdxSchema.BasicProfile);
    setBasicProfile(resp);
  } catch (error) {
    setError(error);
  } finally {
    setFetching(false);
  }
};


```

Here we also use idx instance and call get method on it  providing just `basicProfile` as an argument. Since we are authenticated ceramic  client is aware of our DID so we do not have to pass it in. However, we can ready basicProfile data about any DID since this data is public. In order to read basicProfile for DID that is not ours we have to explicitly pass that DID as a second parameter.

