---
title: "An adventure in automated testing with Jest and Mongoose"
description: "Beginner pitfalls with building automated tests that compare JS objects with mongoose objects."
date: "2022-02-27"
timezone: "Europe/London"
utterances_term: "Hello World"
categories: ["Jest", "Mongoose", "MongoDB"]
---

# An adventure in automated testing with Jest and Mongoose

> Beginner pitfalls with building automated tests that compare JS objects with mongoose objects.

With thanks to [Jon Sharpe](https://stackoverflow.com/users/3001761/jonrsharpe), who kindly pointed me to the problem and solution here.
<hr />

## TLDR
#### Mongoose dates != JS Dates
#### Mongoose objects != JS objects. Use 
```js
expect(mongooseObject.toJSON()).toMatchObject(jsObject)
```
#### If you have dates, fix up your `mongooseObject.toJSON()` by replacing any date properties with 
```js
mongooseObjectJson.dateProperty = mongooseObject.dateProperty.toJSON()
```
#### Keep your dependencies up to date.  
 - The older version of Jest we were using hid some diffs, making the issue and solution harder to find.
<hr />

## The long version:
One of my first forays into JavaScript (well, into anything other than VBA and PowerShell) required some automated testing with Jest.  We needed to change the behavior of an object derived from a MongoDB document.  In order to establish the test scenario, I first needed to build tests that confirm creation the parent document works as expected.  
I had never TDD'd before, or done any automated testing at all, but fortunately there was a chunky codebase with lots of examples to guide me. There were still a lot of rookie mistakes!


A little context:

`StepsContext.Create(data)`:
- creates a new Step with a [mongoose.js](https://mongoosejs.com/) model
- saves it to the MongoDB collection
- returns the new Step

So, let's check that works:
```js
describe('Step functions', () => {
    it('should be able to create a step', async () => {
      const step = await StepsContext.create(stepBody)
      expect(step).toEqual(stepBody)
    })
})
```

Well, that's a **fail**.

Hmm, we're getting back more fields than we put in because, of course, we're getting back the version and creation data from Mongo.

So, let's allow for extra fields:
```js
describe('Step functions', () => {
    it('should be able to create a step', async () => {
      const step = await StepsContext.create(stepBody)
      expect(step).toMatchObject(stepBody)
    })
})
```

All good now! **No?**

```console
 FAIL  __tests__/application_process/steps/delete.js
  Step functions
    ✕ should be able to create a step (158 ms)

  ● Step functions › should be able to create a step

    expect(received).toMatchObject(expected)

    - Expected  - 0
    + Received  + 4
```

Oh, ok, we just don't know how to use Jest's [expect.toMatchObject](https://jestjs.io/docs/expect#tomatchobjectobject) properly.  Right?

Let's build a Minimum Reproducible Example to confirm:
```js
describe('Step functions', () => {
    it('should match a partial object', ()=>{
        const received = { abc:'def', hij:'klm' }
        const expected = { abc:'def' }
        expect(received).toMatchObject(expected)
    }),
    it('should be able to create a step', async () => {
      const step = await StepsContext.create(stepBody)
      expect(step).toMatchObject(stepBody)
    })
})
```

Oh dear, the partial object experiment passes, but our Steps test still fails:
```console
FAIL  __tests__/application_process/steps/delete.js
  Step functions
    ✓ should match a partial object (1 ms)
    ✕ should be able to create a step (158 ms)

  ● Step functions › should be able to create a step

    expect(received).toMatchObject(expected)

    - Expected  - 0
    + Received  + 4
```

Something is not as it seems!

Let's experiment a bit more.  What about if we just target a known property of a successfully created document, and check that it's there on the return:

```js
describe('Step functions', () => {
    it('should be able to create a step', async () => {
      const step = await StepsContext.create(stepBody)
      expect(step._id).toHaveProperty('_bsontype', 'ObjectID')
    }
}
```
Well, that works.  ***And it fails when our input data fails validation***.  Great!  This doesn't feel good though.  

We aren't really testing that our object is being stored correctly, we're just testing that **something** is being stored.


<br>
> First issue is that we were using an old version of Jest, which seemed to have a bug.  it was actually failing on the Mongoose date fields not matching the JS object's date strings, but was reporting that it was failing on the extra props.

***Lesson eventually learned: keep dependencies up to date.***

<br>



We have our MRE, so let's do some more research, and then ask Stack Overflow.

Fortunately the masterful [Textbook / Jon Sharpe](https://stackoverflow.com/users/3001761/jonrsharpe) could point me to an answer.  No doubt shocked that I'd turned up with an MRE this time, rather than a typical cry of "Help!  It doesn't work!", he kindly discovered and pointed me to another poor soul stuck in the same problem: https://stackoverflow.com/q/61217935/15991582 .

### You cannot simply compare a mongoose object with a vanilla JS object with the same enumerable properties.

Experienced readers will know this, but for those more my level, try:
```js
console.log(new MongooseModel(data).save())
```

There's a lot there.  No wonder it didn't match.

Fortunately there's a solution.  Use the Mongoose object's `.toJSON()` method to clean things up:



```js
describe('Step functions', () => {
    it('should be able to create a step', async () => {
      const step = await StepsContext.create(stepBody)
      expect(step.toJSON()).toMatchObject(stepBody)
    })
})
```

That gets us closer!  Great!

Unfortunately, the date properties are still not converted to JS `Date`s or strings, so there's a little more to do.  We could get smart and do this by looping through the Object.Keys(stepJSON), but this is a test, so let's keep things simple:


```js
describe('Step functions', () => {
  it('should create a step', async () => {
    const step = await StepsContext.create(stepBody)
    // return from .create() matches stepdata content
    const stepJsObject = step.toJSON()
    stepJsObject.eligibilityFromDate = step.eligibilityFromDate.toJSON()
    stepJsObject.eligibilityToDate = step.eligibilityToDate.toJSON()
    expect(stepJsObject).toMatchObject(stepBody)
    // return from .create() has a mongoDB ID
    expect(step._id).toHaveProperty('_bsontype', 'ObjectID')
  })
}
```

So: [Learn the lessons in the TLDR!](#tldr)

<br>
That stepJsObject doesn't look much like JSON to me, so there' clearly still plenty to learn here, but these tests are working properly now, so let's look deeper into the `.toJSON()` method another time.