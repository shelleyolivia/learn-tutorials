# Introduction

In this tutorial you will learn how to create a simple React app by leveraging the existing subgraphs. It includes configuring GraphQL queries and simple data visualization. 


# Requirements

You will need to have `Node.js >= 14.0.0` and `npm >= 5.6` on your machine.

# Project setup

Run the following command to create a new project.

```bash
npx create-react-app my-app
cd my-app
```

Also, you need to install some packages and run the app at localhost.
```bash
npm i @apollo/client graphql echarts-for-react echarts
npm start
```

You should see similar message in your terminal.
```
Compiled successfully!

You can now view my-app in the browser.

  Local:            http://localhost:3000
  On Your Network:  http://192.168.0.145:3000

Note that the development build is not optimized.
To create a production build, use yarn build.
```
Then, you are able to see your project at http://localhost:3000 by default.

# Create Query Client

Create `QueryClients.js` under `src` folder. Add following script to configure a query client to interact with [HOPR Subgraph Mainnet](https://thegraph.com/legacy-explorer/subgraph/minatofund/hopr-subgraph-mainnet). You need this client to send GraphQL query to the subgraph endpoint.

```JavaScript
import { ApolloClient, InMemoryCache, gql } from '@apollo/client'

export const hoprClient = new ApolloClient({
    uri: 'https://api.thegraph.com/subgraphs/name/minatofund/hopr-subgraph-mainnet',
    cache: new InMemoryCache()
  })

```

# Configure Query and Visualization

Crete `HolderBalance.js` under `src` folder. Add the script below.

```JavaScript
import React from 'react'
import ReactECharts from 'echarts-for-react'
import { gql } from '@apollo/client'
import {hoprClient} from './QueryClients'

class HolderBalance extends React.Component {
  state = {
    loading: true,
    holderBalance: []
  }

  async fetchData() {
    const holderBalanceQuery = `
    query accounts($skip: Int!) {
        accounts(where:{HoprBalance_gt:0, id_not:"0x0000000000000000000000000000000000000000"},first:1000, skip:$skip, orderBy: HoprBalance, orderDirection: desc)
        {
          totalBalance
        }
      }
    `

    try {
      let skip = 0
      let allResults = []
      let found = false
      while (!found) {
        let result = await hoprClient.query({
          query: gql(holderBalanceQuery),
          variables: {
            skip: skip
          },
          fetchPolicy: 'cache-first',
        })
        allResults = allResults.concat(result.data.accounts)
        if (result.data.accounts.length < 1000) {
          found = true
        } else {
          skip += 1000
        }
      }

      return allResults
    } catch (e) {
      console.error(e)
    }
  }

  componentDidMount() {
    this.fetchData().then(data => {
      this.processHolderBalanceData(data)
    }).catch(e => {
      console.error(e)
    })
  }

  processHolderBalanceData(data) {
    //Sizes are sorted in descending order
    const sizes = data.map(item => (parseFloat(item.totalBalance)))
    const spliters = [10, 100, 1000, 10000, 100000]
    //Find indexes spliting the sizes by given spliters
    let spliterIndex = spliters.length - 1
    let splitingIndexes = [0]
    for (let [idx, size] of sizes.entries()) {
      while (size <= spliters[spliterIndex]) {
        splitingIndexes.push(idx)
        spliterIndex -= 1
      }
      if (spliterIndex < 0 )
        break
    }
    //Split sizes, get chunk size and calculate sum of each chunk
    const chunkNames = ['>100k 🐳', '10k-100k', '1k-10k', '100-1k', '10-100', '0-10']
    let splited = []
    for (let i = 0; i < splitingIndexes.length - 1; i++) {
      const chunk = sizes.slice(splitingIndexes[i], splitingIndexes[i + 1])
      splited.push({
        name: chunkNames[i],
        chunk: chunk
      })
    }
    splited.push({
      name: chunkNames[splitingIndexes.length - 1],
      chunk: sizes.slice(splitingIndexes[splitingIndexes.length - 1])
    })
    //Update holder balance data
    const reducer = (accumulator, currentValue) => accumulator + currentValue
    let holderBalance = splited.map((item) => {
      const length = item.chunk.length
      const noun = length === 1 ? 'holder' : 'holders'
      return ({
        name: `${item.name} (${length} ${noun})`,
        value: item.chunk.reduce(reducer)
      })
    })
    this.setState({
      loading: false,
      holderBalance: holderBalance
    })
  }

  chart() {
    if (this.state.loading) {
      return <p>Loading...</p>
    }
    const options = {
      tooltip: {
          trigger: 'item',
          formatter: '{a} <br/>{b} : {c} ({d}%)'
      },
      // legend: {
      //   top: '5%',
      //   left: 'center'
      // },
      series: [
        {
          name: 'Holder Balance',
          type: 'pie',
          radius: '50%',
          emphasis: {
            itemStyle: {
              shadowBlur: 10,
              shadowOffsetX: 0,
              shadowColor: 'rgba(0, 0, 0, 0.5)'
            }
          },
          data: this.state.holderBalance.map((item) => {
            return {
              ...item,
              value: parseFloat(item.value.toFixed(2))
            }
          }),
        }
      ]
    }
    return <ReactECharts option={options} />;
  }

  render() {
    return (
      <div>
        {this.chart()}
      </div>
    )
  }
}

export default HolderBalance
```

Let me explain the functionality of each part of the code.

Import necessary components. 
```JavaScript
import React from 'react'
import ReactECharts from 'echarts-for-react'
import { gql } from '@apollo/client'
import {hoprClient} from './QueryClients'

```

Define the class and state.
```JavaScript
class HolderBalance extends React.Component {
  state = {
    loading: true,
    holderBalance: []
  }
```

Configure the query you are going to send. Leave `skip` as a variable because you need to adjust it at runtime. This query means we would like to get accounts' HOPR balance and exclude 0x0 address.
```JavaScript
 async fetchData() {
    const holderBalanceQuery = `
    query accounts($skip: Int!) {
        accounts(where:{HoprBalance_gt:0, id_not:"0x0000000000000000000000000000000000000000"},first:1000, skip:$skip, orderBy: HoprBalance, orderDirection: desc)
        {
          totalBalance
        }
      }
    `
```

Use a while loop to send query until the result has less than 1000 records. This is because a single query can only return 1000 results as maximum. In order to get all data, you need to adjust the `skip` variable.
```JavaScript
    try {
      let skip = 0
      let allResults = []
      let found = false
      while (!found) {
        let result = await hoprClient.query({
          query: gql(holderBalanceQuery),
          variables: {
            skip: skip
          },
          fetchPolicy: 'cache-first',
        })
        allResults = allResults.concat(result.data.accounts)
        if (result.data.accounts.length < 1000) {
          found = true
        } else {
          skip += 1000
        }
      }

      return allResults
    } catch (e) {
      console.error(e)
    }
  }

  componentDidMount() {
    this.fetchData().then(data => {
      this.processHolderBalanceData(data)
    }).catch(e => {
      console.error(e)
    })
  }
```

Now you have the raw data, and the next part is to visualize it. In this section, we split the accounts into different categories by HOPR token balance. 

```JavaScript
  processHolderBalanceData(data) {
    //Sizes are sorted in descending order
    const sizes = data.map(item => (parseFloat(item.totalBalance)))
    const spliters = [10, 100, 1000, 10000, 100000]
    //Find indexes spliting the sizes by given spliters
    let spliterIndex = spliters.length - 1
    let splitingIndexes = [0]
    for (let [idx, size] of sizes.entries()) {
      while (size <= spliters[spliterIndex]) {
        splitingIndexes.push(idx)
        spliterIndex -= 1
      }
      if (spliterIndex < 0 )
        break
    }
```

The categories are '>100k 🐳', '10k-100k', '1k-10k', '100-1k', '10-100', '0-10'.
```JavaScript
    //Split sizes, get chunk size and calculate sum of each chunk
    const chunkNames = ['>100k 🐳', '10k-100k', '1k-10k', '100-1k', '10-100', '0-10']
    let splited = []
    for (let i = 0; i < splitingIndexes.length - 1; i++) {
      const chunk = sizes.slice(splitingIndexes[i], splitingIndexes[i + 1])
      splited.push({
        name: chunkNames[i],
        chunk: chunk
      })
    }
    splited.push({
      name: chunkNames[splitingIndexes.length - 1],
      chunk: sizes.slice(splitingIndexes[splitingIndexes.length - 1])
    })
    //Update holder balance data
    const reducer = (accumulator, currentValue) => accumulator + currentValue
    let holderBalance = splited.map((item) => {
      const length = item.chunk.length
      const noun = length === 1 ? 'holder' : 'holders'
      return ({
        name: `${item.name} (${length} ${noun})`,
        value: item.chunk.reduce(reducer)
      })
    })
    this.setState({
      loading: false,
      holderBalance: holderBalance
    })
  }
```

Finally, you generate the chart and export it.
```JavaScript
  chart() {
    if (this.state.loading) {
      return <p>Loading...</p>
    }
    const options = {
      tooltip: {
          trigger: 'item',
          formatter: '{a} <br/>{b} : {c} ({d}%)'
      },
      // legend: {
      //   top: '5%',
      //   left: 'center'
      // },
      series: [
        {
          name: 'Holder Balance',
          type: 'pie',
          radius: '50%',
          emphasis: {
            itemStyle: {
              shadowBlur: 10,
              shadowOffsetX: 0,
              shadowColor: 'rgba(0, 0, 0, 0.5)'
            }
          },
          data: this.state.holderBalance.map((item) => {
            return {
              ...item,
              value: parseFloat(item.value.toFixed(2))
            }
          }),
        }
      ]
    }
    return <ReactECharts option={options} />;
  }

  render() {
    return (
      <div>
        {this.chart()}
      </div>
    )
  }
}

export default HolderBalance
```

Finally, you can use display the new chart in your app. Please replace the code in `App.js` by the following one. 
```JavaScript
import './App.css';
import HolderBalance from './HolderBalance'

function App() {
  return (
    <div className="App">
      <div> <HolderBalance /> </div>
    </div>
  );
}

export default App;

```

Please save all your changes and refresh the localhost page. You should see the nice pie chart we just created. 

![Graph React App Pie Chart](../../../.gitbook/assets/graph-react-app-pie-chart.png)


# Conclusion

Congratulations! You complete this tutorial. You have learned how to configure GraphQL query to retrieve data from existing subgraph. You also understand how to visualize the data properly on your React app. 

# About the Author

This tutorial was created by [Minato Fund](https://github.com/minatofund), an active group helping merge blockchain projects operating as a validator.
