# Step 4: Fetching a Post

## Steps
* Add API endopoint for getting Arweave entry by transactions id in api/arweave/[transactionId].ts
  * In this endoint get the data for the entry (getData method), get status of the entry (get Status method), get timestamp (get method) and all the tags associated with entry
* Install swr package
* Setup swr by adding a axios fetcher in fetchers folder
* Create a custom hook in useArweave.ts called `useGeetTransaction` which will be responsible for retrieving transaction data (hitting newly created endpoint) by transactionId using swr.
* Create a PostDetails component in which you use above custom hook (transactionId is a prop). Display returned data about Arweave transaction title and body being the most important ones.
* Create a page `/entries/view/[transactionId].tsx` which renders PostDetails component.


## [Concept 1]

## [Concept 2, etc]

# Implementation 🧩

##### _Listing 4.1: Code for fetching a post_
>>>>>>> Insert code here

# Challenge 🏋️

Navigate to `[ ]` in your editor and follow the steps included as comments to finish writing the `[ ]` function. We include a description along with a link to the documentation you need to review in order to implement each line. The relevant code block is also included in [Listing 4.2](#listing-42-instructions-for-fetching-a-post) below.

##### _Listing 4.2: Instructions for fetching a post_
