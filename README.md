# SoFlow

An attempt at making AWS SWF usage practical.


## Usage

### Install package


```shell
# Temporary method for now
docker-compose up --build
docker cp soflow_dev_1:/soflow/compiled ~/Projects/myproject/node_modules/soflow
```

### Define tasks

__tasks/index.js__

```javascript
export addOrderToDatabase from './addOrderToDatabase'
export sendOrderConfirmation from './sendOrderConfirmation'
```

__tasks/addOrderToDatabase.js__
```javascript
async function addOrderToDatabase(data) {
  // do your stuff
  return result
}

addOrderToDatabase.config = {
  SWF: {
    lambda: {
      memorySize: 128,
      timeout: 60,
    }
  }
}
export default addOrderToDatabase
```

__tasks/sendOrderConfirmation.js__
```javascript
async function sendOrderConfirmation(orderData) {
  // do your stuff
}

export default sendOrderConfirmation
```

### Define workflows

__workflows/index.js__
```javascript
export CreateOrder from './CreateOrder'
```

__workflows/CreateOrder.js__
```javascript
async function CreateOrder({
  input: {
    confirmationEmailRecipient,
    products,
    customerId,
  },
  actions,
  tasks: {
    addOrderToDatabase,
    sendOrderConfirmation,
  }
}) {
  const orderData = await addOrderToDatabase(customerId, products)
  await sendOrderConfirmation(orderData)

  return orderData
}

CreateOrder.config = {
  SWF: {
    tasks: {
      addOrderToDatabase: {type: 'lambda'}, // default
      sendOrderConfirmation: {type: 'activityTask'}
    }
  }
}

export default CreateOrder
```

### Deploy
```javascript
import {SWF} from 'soflow'

SWF.deploy({
  namespace: 'myapp-production',
  domain: 'myapp',
  version: 'v1',
  workflowsPath: 'workflows', // default
  tasksPath: 'tasks',         // default
  files: [
    'package.json',
    'node_modules/**',
    'workflows/**',
    'tasks/**',
  ],
  runDeciderImmediately: false, // default
})
```

### Execute workflow
```javascript
import {SWF} from 'soflow'

async function runWorkflow() {
  const result = await SWF.executeWorkflow({
    domain: 'myapp',
    namespace: 'myapp-production',
    id: 'CreateOrder-12345',
    type: 'CreateOrder',
    version: 'v1',
    input: {productIds: [1,2,3]},
  })
}
```

## Development

### Getting started


```shell
# Continuously build files and run unit tests
docker-compose up --build
```

### Running integration tests
```shell
# Copy docker-compose override file and configure AWS credentials
cp docker-compose.override.example.yml docker-compose.override.yml

# Run tests
docker-compose exec dev devops/integration-tests

# Run tests without tearing down CloudFormation stack
docker-compose exec dev devops/integration-tests --no-teardown

# Run tests without setting up and tearing down CloudFormation stack
docker-compose exec dev devops/integration-tests --no-setup --no-teardown
```
