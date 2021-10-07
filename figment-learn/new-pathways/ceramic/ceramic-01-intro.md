# Intro

## What is Ceramic?
Ceramic is a platform for creating, hosting and sharing streams of data in decentralized way.

## Why do we need it?
Most of the information on today’s internet is siloed out in databases used by specific applications. All of this information is not easily accessible for the outside world. If you don’t have direct access to database of if there is no API exposed by the application, you might need to go and look around somewhere else. This results in data being duplicated and locked away in many different databases. What if there was one place where you could get all the relevant content you need? What if content was open-sources the same way code is open-sourced using GitHub? That’s exactly the promise of Ceramic. Open-source content which can be easily shared between applications using just one SDK rather than dozens of different external APIs.


## What is IDX?
Two main concepts whiten Ceramic network are Streams and StreamTypes. Streams are nothing more than append only log of commits stored on IPFS. StreamTypes on the other hand are functions which are applied to those logs. One other important responsibility of StreamTypes is authorisation of users to perform write operations to stream.  Different StreamTypes might need different levels of authorisation. One of the authentication mechanisms that StreamTypes supports is DIDs, the W3C standard for decentralized identifiers. DIDs are used to associate globally-unique, platform-agnostic string identifier with a DID document where all the data required for signature verification and encryption is stored. This DID is essential to another protocol build on top of Ceramic Called IDX. IDX is a protocol for decentralized identity that makes it easy to store all user-generated data from different apps in one place. With IDX there no more need for juggling many API Keys for different external services to get access to data generate by users on different applications.

## Why do we need it?

IDX makes it easy to have one identity and use it with different applications. It means that there is just one “users table” which is not stored in one database but rather spread across the Ceramic network. With IDX users can own their identities and data.

## Key concepts
Stream - basic building block on Ceramic network. It represents append-only log of commits for each piece of information. Streams are stored on IPFS.

StreamID - It is an immutable identifier for a stream. StreamID does not change even when new data gets added to IPFS.

StreamTypes - These are functions which are applied to information coming in to stream also known as commits. Every stream has to specify StreamType




