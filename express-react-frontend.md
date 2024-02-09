## Express React Build Pt 2

In this build we will

- Build an Express API
- Use Mongo/Mongoose with 1 model
- Build a Full Crud Frontend with React

# React People Setup, Index, Create

## Setup

- open terminal in frontend folder

- install react router `npm install react-router-dom`


## Installing Router

- Update index.js to like like so

```js
import React from "react";
import ReactDOM from "react-dom";
// IMPORT ROUTER
import { BrowserRouter as Router } from "react-router-dom";
import App from "./App";
import reportWebVitals from "./reportWebVitals";

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <Router>
      <App />
    </Router>
  </React.StrictMode>
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();
```

## Scoping Out Our Components

- Create a components and pages folder

- In the components folder create a Header.js and Main.js file

- In the pages folder create a Index.js, Show.js, and Home.js

- Write the component boilerplate and export the component in all the created files

```jsx
const Component = (props) => {
    return <h1>Component Name</h1>
}

export default Component
```

## App.js

Our desired component Architecture

```
-> App
  -> Header
  -> Main |state: people|
    -> Routes
      -> Route |path: "/"|
        -> Home |Props: people, createPeople|
      -> Route |path: "/people"|
        -> Index |Props: people|
      -> Route |path="/people/:id|
        -> Show |Props: people, updatePeople, deletePeople|
```

Let's add the following to App.js

```js
import "./App.css";
import Header from "./components/Header";
import Main from "./components/Main";

function App() {
  return (
    <div className="App">
      <Header />
      <Main />
    </div>
  );
}

export default App;
```

## Setting up router in Main.js

- let's create our routes

```js
import { useEffect, useState } from "react";
import { Route, Routes } from "react-router-dom";
import Index from "../pages/Index";
import Show from "../pages/Show";
import Home from "../pages/Home";

const Main = (props) => {
  return (
    <main>
      <Routes>
        <Route path="/" element={<Home />}/>
        <Route path="/people" element={<Index >}/>
        <Route path="/people/:id" element={<Show/>}/>
      </Routes>
    </main>
  )
}

export default Main;
```

## Setting Up Navigation

First, lets pick a css library and style with it! It will be easier to style as we go rather than change everything at the end to fit our CSS. Let use React styling components from Bulma! 

- install `npm i react-bulma-components` [Check out their docs!](https://couds.github.io/react-bulma-components/?path=/story/welcome--page)


Let's put the following in Header.js

```js
import { Link } from "react-router-dom";
import { Navbar } from "react-bulma-components";

const Header = () => {
  return (
      <Navbar display="flex" justifyContent="space-evenly" fixed="top" size="large">
        <Navbar.Container>
            <Navbar.Item>
                <Link to="/">
                    Home
                </Link>
            </Navbar.Item>
        </Navbar.Container>
        <Navbar.Container>
            <Navbar.Item>
                <Link to="/people">
                    People
                </Link>
            </Navbar.Item>
        </Navbar.Container>
      </Navbar>
  );
}

export default Header;
```

## Displaying People in Index

We need the state to exist in Main so it can be shared between Index, Home, and Show. So let's update Main to have:

- state to hold our list of people
- function to make the api call for people
- function to create a new person
- useEffect to make initial call for people list
- pass the people state and the create function to Home and the people state to Index

Main.js

```js
import { useEffect, useState } from "react";
import { Route, Routes } from "react-router-dom";
import Index from "../pages/Index";
import Show from "../pages/Show";
import Home from "../pages/Home";

const Main = (props) => {
  const [people, setPeople] = useState(null);

  const URL = "http://localhost:4000/people/";

  const getPeople = async () => {
    const response = await fetch(URL);
    const data = await response.json();
    setPeople(data.data);
  };

  const createPeople = async (person) => {
    // make post request to create people
    await fetch(URL, {
      method: "post",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(person),
    });
    // update list of people
    getPeople();
  };

  useEffect(() => {
    getPeople()
  }, []);

  return (
    <main>
      <Routes>
        <Route path="/" element={<Home people={people} createPeople={createPeople}/>}/>
        <Route path="/people" element={<Index people={people} />}/>
        <Route path="/people/:id" element={<Show/>}/>
      </Routes>
    </main>
  );
}

export default Main;
```

Let's now display the people in Index.js and add some styling components along the way from Bulma

```js
import {Content, Button} from 'react-bulma-components'
import {Link} from "react-router-dom"

const Index = (props) => {
    // loaded function
    const loaded = () => {
        console.log(props.people)
        return props.people.map((person) => (
            <Content 
                display="flex" 
                flexDirection="row" 
                justifyContent="space-evenly"
                alignItems="baseline" 
                className="has-background-grey-lighter person-content">
                <h3>{person.firstName} {person.lastName}</h3>
                <p>{person.streetAddressL1}</p>
                {person.streetAddress2 ? <p>{person.streetAddress2}</p>: ""}
                <p>{person.city}, {person.state} {person.zip}</p>
                <Link to={`/people/${person._id}`}>
                    <Button color='text'>
                        See More
                    </Button>
                </Link>
            </Content>
        ));
    };
  
    const loading = () => {
      return <h1>Loading...</h1>;
    };
    return (props.people ? loaded() : loading());
  }

export default Index
```

## Creating People

Let's now add a form to our Home.js

- state to hold the form data
- form inputs in our JSX
- handlechange function to allow our state to control the form
- handlesubmit function handle form submisssion

```js
import {useState} from 'react'
import { Box, Form, Button } from "react-bulma-components";

const Home = (props) => {
    const newForm = {
        firstName: "",
        lastName: "",
        streetAddressL1: "",
        streetAddressL2: "",
        city: "",
        state: "",
        zip: "",
        country: "",
        email: "",
        phoneNumber: "",
      }
    // state to hold formData
    const [form, setForm] = useState(newForm);
    // destructuring Form object for ease of use
    const { Input, Field, Label } = Form;

    // handleChange function for form
    const handleChange = (e) => {
        setForm({ ...form, [e.target.name]: e.target.value });
    };

  // handle submit function for form
  const handleSubmit = (e) => {
    e.preventDefault();
    props.createPeople(form);
    setForm(newForm);
  };

  const loaded = () => {
    return(
        <section>
            <Box className='form-box'>
            <h2 className='is-size-3 has-font-weight-bold'>Create Person</h2>
            <form onSubmit={handleSubmit}>
                <Field>
                    <Label>First Name</Label>
                    <Input 
                      name="firstName" 
                      value={form.firstName}
                      placeholder='First Name' 
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>Last Name</Label>
                    <Input 
                      name="lastName" 
                      value={form.lastName}
                      placeholder='Last Name'  
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>Street Address Line 1</Label>
                    <Input 
                      name="streetAddressL1" 
                      value={form.streetAddressL1} 
                      placeholder="'567 Rainbow Bridge Dr'" 
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>Line 2</Label>
                    <Input 
                      name="streetAddressL2" 
                      value={form.streetAddressL2} 
                      placeholder="'Apt #4'" 
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>City</Label>
                    <Input 
                      name="city" 
                      value={form.city} 
                      placeholder='City' 
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>State</Label>
                    <Input 
                      name="state" 
                      value={form.state} 
                      placeholder="State"
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>Zipcode</Label>
                    <Input 
                      name="zip" 
                      value={form.zip} 
                      placeholder='Zipcode'
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>Email</Label>
                    <Input 
                      name="email" 
                      value={form.email}
                      placeholder='Email' 
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>Phone Number</Label>
                    <Input 
                      name="phoneNumber" 
                      value={form.phoneNumber} 
                      placeholder='(xxx)-xxx-xxxx'
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Button color="primary">
                    Submit
                </Button>
                </form>
            </Box>
        </section>
    )
  }

  const loading = () => {
    return <h1>Loading...</h1>;
  };

  return (props.people ? loaded() : loading());
}

export default Home
```

You should now be able to see all the people and create people

# People Build, Show, Edit and Delete

## The Show Page

Let's pass the people data to the Show page via props and make a update and delete function for the show page, head over to Main.js

```js
import { useEffect, useState } from "react"
import { Route, Routes } from 'react-router-dom'
import Index from "../pages/Index"
import Show from '../pages/Show'

const Main = (props) => {
  const [people, setPeople] = useState(null)

  const URL = 'http://localhost:4000/people'

  //fetches all people from our API backend
  const getPeople = async () => {
    const reponse = await fetch(URL)
    const data = await reponse.json()
    setPeople(data.data)
  }

  const createPeople = async (person) => {
    //make post request to create people
    await fetch(URL, {
        method: "POST",
        headers: {
            "Content-Type": "application/json"
        },
        body: JSON.stringify(person),
    })
    // update our components list of people
    getPeople()
  }

  const updatePeople = async (person, id) => {
    // make post request to create people
    await fetch(URL + id, {
      method: "PUT",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(person),
    });
    // update list of people
    getPeople();
  };
  
  const deletePeople = async (id) => {
    // make post request to create people
    await fetch(URL + id, {
        method: "DELETE",
    });
    // update list of people
    getPeople();
  };

  useEffect(() => {
      getPeople()
  }, []);

  return (
    <main>
      <Routes>
        <Route path="/" element={<Home people={people} createPeople={createPeople}/>}/>
        <Route path="/people" element={<Index people={people} />}/>
        <Route path="/people/:id" element={<Show people={people} updatePeople={updatePeople} deletePeople={deletePeople}/>}/>
      </Routes>
    </main>
  )
}

export default Main
```

Let's grab the selected person from the people array in props and display them. We will also continue to style as we go! 

Show.js

```js
import {useParams} from "react-router-dom"
import {Card} from 'react-bulma-components'

const Show = (props) => {
  const params = useParams()
  const id = params.id;
  const people = props.people;
  const person = people.find((p) => p._id === id);

    return(
        <Card textAlign='center' style={{width:'400px', margin:'0 auto'}}>
        <Card.Content>
            <Card.Header.Title className="is-size-3">
                {person.firstName} {person.lastName}
            </Card.Header.Title>
                <Card.Content>
                    <strong>Phone Number: </strong><p>{person.phoneNumber}</p>
                    <strong>Address: </strong>
                    <p>{person.streetAddressL1}</p>
                    {person.streetAddress2 ? <p>{person.streetAddress2}</p>: ""}
                    <p>{person.city}, {person.state} {person.zip}</p>
                    <p>{person.country}</p>
                    <strong>Email: </strong><p>{person.email}</p>
                </Card.Content>
            </Card.Content>
        </Card>
    )
}

export default Show
```

## Updating a Person

On the show page let's add

- state for a form

- handleChange and handleSubmit function

- a form in the JSX below the person

```js
import {useState} from 'react'
import {useParams} from "react-router-dom"
import {Card} from 'react-bulma-components'
import { Box, Form, Button } from "react-bulma-components";


const Show = (props) => {
  const params = useParams()
  const id = params.id;
  const people = props.people;
  const person = people.find((p) => p._id === id);

    const newForm = {
        firstName: "",
        lastName: "",
        streetAddressL1: "",
        streetAddressL2: "",
        city: "",
        state: "",
        zip: "",
        country: "",
        email: "",
        phoneNumber: "",
    }
    // state to hold formData
    const [form, setForm] = useState(person);
    // destructuring Form object for ease of use
    const { Input, Field, Label } = Form;

    // handleChange function for form
    const handleChange = (e) => {
        setForm({ ...form, [e.target.name]: e.target.value });
    };

    const handleSubmit = (event) => {
    event.preventDefault();
    props.updatePeople(form, person._id);
    navigate("/");
    };

    return(
        <>
        <Card textAlign='center' style={{width:'400px', margin:'0 auto'}}>
        <Card.Content>
            <Card.Header.Title className="is-size-3">
                {person.firstName} {person.lastName}
            </Card.Header.Title>
                <Card.Content>
                    <strong>Phone Number: </strong><p>{person.phoneNumber}</p>
                    <strong>Address: </strong>
                    <p>{person.streetAddressL1}</p>
                    {person.streetAddress2 ? <p>{person.streetAddress2}</p>: ""}
                    <p>{person.city}, {person.state} {person.zip}</p>
                    <p>{person.country}</p>
                    <strong>Email: </strong><p>{person.email}</p>
                </Card.Content>
            </Card.Content>
        </Card>
        <section>
            <Box className='form-box'>
            <h2 className='is-size-3 has-font-weight-bold'>Edit Person</h2>
            <form onSubmit={handleSubmit}>
                <Field>
                    <Label>First Name</Label>
                    <Input 
                      name="firstName" 
                      value={form.firstName}
                      placeholder='First Name' 
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>Last Name</Label>
                    <Input 
                      name="lastName" 
                      value={form.lastName}
                      placeholder='Last Name'  
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>Street Address Line 1</Label>
                    <Input 
                      name="streetAddressL1" 
                      value={form.streetAddressL1} 
                      placeholder="'567 Rainbow Bridge Dr'" 
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>Line 2</Label>
                    <Input 
                      name="streetAddressL2" 
                      value={form.streetAddressL2} 
                      placeholder="'Apt #4'" 
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>City</Label>
                    <Input 
                      name="city" 
                      value={form.city} 
                      placeholder='City' 
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>State</Label>
                    <Input 
                      name="state" 
                      value={form.state} 
                      placeholder="State"
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>Zipcode</Label>
                    <Input 
                      name="zip" 
                      value={form.zip} 
                      placeholder='Zipcode'
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>Email</Label>
                    <Input 
                      name="email" 
                      value={form.email}
                      placeholder='Email' 
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>Phone Number</Label>
                    <Input 
                      name="phoneNumber" 
                      value={form.phoneNumber} 
                      placeholder='(xxx)-xxx-xxxx'
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Button color="primary">
                    Submit
                </Button>
                </form>
            </Box>
        </section>
        </>
    )
}

export default Show
```

## Deleting a Person

Last Stop is adding a button on the show page to delete a user!

```js
import {useState} from 'react'
import {useParams, useNavigate} from "react-router-dom"
import {Card} from 'react-bulma-components'
import { Box, Form, Button } from "react-bulma-components";


const Show = (props) => {
  const params = useParams()
  const navigate = useNavigate()

  const id = params.id;
  const people = props.people;
  const person = people.find((p) => p._id === id);

    const newForm = {
        firstName: "",
        lastName: "",
        streetAddressL1: "",
        streetAddressL2: "",
        city: "",
        state: "",
        zip: "",
        country: "",
        email: "",
        phoneNumber: "",
    }
    // state to hold formData
    const [form, setForm] = useState(person);
    // destructuring Form object for ease of use
    const { Input, Field, Label } = Form;

    // handleChange function for form
    const handleChange = (e) => {
        setForm({ ...form, [e.target.name]: e.target.value });
    };

    const handleSubmit = (event) => {
        event.preventDefault();
        props.updatePeople(form, person._id);
        navigate("/");
      };
    
      const removePerson = (e) => {
        e.preventDefault()
        props.deletePeople(person._id);
        navigate("/");
      };


    return(
        <>
        <Card textAlign='center' style={{width:'400px', margin:'0 auto'}}>
        <Card.Content>
            <Card.Header.Title className="is-size-3">
                {person.firstName} {person.lastName}
            </Card.Header.Title>
                <Card.Content>
                    <strong>Phone Number: </strong><p>{person.phoneNumber}</p>
                    <strong>Address: </strong>
                    <p>{person.streetAddressL1}</p>
                    {person.streetAddress2 ? <p>{person.streetAddress2}</p>: ""}
                    <p>{person.city}, {person.state} {person.zip}</p>
                    <p>{person.country}</p>
                    <strong>Email: </strong><p>{person.email}</p>
                </Card.Content>
            </Card.Content>
        </Card>
        <Button color="danger" onClick={removePerson}>
            Delete
        </Button>
        <section>
            <Box className='form-box'>
            <h2 className='is-size-3 has-font-weight-bold'>Edit Person</h2>
            <form onSubmit={handleSubmit}>
                <Field>
                    <Label>First Name</Label>
                    <Input 
                      name="firstName" 
                      value={form.firstName}
                      placeholder='First Name' 
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>Last Name</Label>
                    <Input 
                      name="lastName" 
                      value={form.lastName}
                      placeholder='Last Name'  
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>Street Address Line 1</Label>
                    <Input 
                      name="streetAddressL1" 
                      value={form.streetAddressL1} 
                      placeholder="'567 Rainbow Bridge Dr'" 
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>Line 2</Label>
                    <Input 
                      name="streetAddressL2" 
                      value={form.streetAddressL2} 
                      placeholder="'Apt #4'" 
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>City</Label>
                    <Input 
                      name="city" 
                      value={form.city} 
                      placeholder='City' 
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>State</Label>
                    <Input 
                      name="state" 
                      value={form.state} 
                      placeholder="State"
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>Zipcode</Label>
                    <Input 
                      name="zip" 
                      value={form.zip} 
                      placeholder='Zipcode'
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>Email</Label>
                    <Input 
                      name="email" 
                      value={form.email}
                      placeholder='Email' 
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Field>
                    <Label>Phone Number</Label>
                    <Input 
                      name="phoneNumber" 
                      value={form.phoneNumber} 
                      placeholder='(xxx)-xxx-xxxx'
                      onChange={(e) => {handleChange(e)}}/>
                </Field>
                <Button color="primary">
                    Submit
                </Button>
                </form>
            </Box>
        </section>
        </>
    )
}

export default Show
```

That's it for our super basic full CRUD app! 

Or... is it? 

# BONUS: Loading, Error, Lifting State

## Loading and Error Pages

- create a page called `Loading` and another page called `Error`

- give them both a basic component structure

```jsx
const Component = () => {
    return(
        <h1>Component Name<h1>
    )
}

export default Component
```

- feel free to use whatever you want for these pages, but we'll be using a third party library for our loading spinner

- for our error page, we will be importing an image - SVG - but the concept is the same for any image in CRA

- install spinner library `npm i react-loader-spinner`

Loading.js

```jsx
import { MagnifyingGlass } from "react-loader-spinner"

const Loading = () => {
    return (
        <div>
            <MagnifyingGlass
                visible={true}
                height="80"
                width="80"
                ariaLabel="MagnifyingGlass-loading"
                wrapperClass="MagnifyingGlass-wrapper"
                glassColor = '#c0efff'
                color = '#e15b64'
            />
        </div>
        )
}

export default Loading
```

- replace `loading()` with our new component
`return (props.people ? loaded() : <Loading />);`

- import the Loading component too as needed (Home.js and Index.js)
`import Loading from '../pages/Loading`

- you might catch a glimpse of the new loader when the page loads, but just to see, test it by just returning the component: `return(<Loading />)` instead of the ternary. **Dont forget to change it back if you do so!**

## Error Page

- you can put whatever you'd like and this can be a catch all error page for any errors you may encounter as a user. Consider how you can make this dynamic in the future.

- copy the following: 

```svg
    <?xml version="1.0" encoding="UTF-8"?>
<!-- Uploaded to: SVG Repo, www.svgrepo.com, Generator: SVG Repo Mixer Tools -->
<svg width="800px" height="800px" viewBox="0 0 73 73" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
    
    <title>design-and-ux/error-handling</title>
    <desc>Created with Sketch.</desc>
    <defs>

</defs>
    <g id="design-and-ux/error-handling" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
        <g id="container" transform="translate(2.000000, 2.000000)" fill="#FFFFFF" fill-rule="nonzero" stroke="#000000" stroke-width="2">
            <rect id="mask" x="-1" y="-1" width="71" height="71" rx="14">

</rect>
        </g>
        <g id="ant" transform="translate(14.000000, 18.000000)" fill-rule="nonzero">
            <g id="Group" transform="translate(3.164062, 14.677734)" fill="#414141">
                <path d="M30.5189648,1.94616211 C24.9732422,1.94616211 20.629248,4.87836914 20.629248,8.62163086 C20.629248,12.3648926 24.9733301,15.2971875 30.5189648,15.2971875 C36.0646875,15.2971875 40.4086816,12.3649805 40.4086816,8.62163086 C40.4087695,4.87836914 36.0646875,1.94616211 30.5189648,1.94616211 Z" id="Shape">

</path>
                <path d="M27.5276074,19.6917188 L23.0559082,8.51238281 C23.5714746,7.66494141 23.8686328,6.67063477 23.8686328,5.60830078 C23.8686328,2.51824219 21.3546973,0.00421875 18.2645508,0.00421875 C15.1744043,0.00421875 12.6604687,2.5181543 12.6604687,5.60830078 C12.6604687,6.49256836 12.8670996,7.32928711 13.2334277,8.07380859 L12.2153906,10.6187695 L7.92966797,8.4759082 L3.4434668,19.6917188 L0.0502734375,19.6917188 L0.0502734375,22.3284375 L5.22852539,22.3284375 L9.31359375,12.1157227 L13.5993164,14.258584 L15.1872363,10.2885645 C16.0711523,10.8717188 17.1285645,11.2123828 18.2644629,11.2123828 C19.260791,11.2123828 20.1963867,10.9498535 21.0079687,10.4921191 L25.7424609,22.3284375 L30.7198828,22.3284375 L30.7198828,19.6917188 L27.5276074,19.6917188 Z" id="Shape">

</path>
            </g>
            <path d="M30.6916699,34.3694531 L26.2199707,23.1901172 C26.7355371,22.3426758 27.0326953,21.3483691 27.0326953,20.2860352 C27.0326953,17.1959766 24.5187598,14.6819531 21.4286133,14.6819531 C21.4286133,18.1013379 21.4286133,22.5837598 21.4286133,25.8901172 C22.4249414,25.8901172 23.3605371,25.6275879 24.1721191,25.1698535 L28.9066113,37.0061719 L33.8840332,37.0061719 L33.8840332,34.3694531 L30.6916699,34.3694531 Z" id="Shape" fill="#000000">

</path>
            <polygon id="Shape" fill="#000000" points="13.8695801 7.43748047 11.0393262 2.72039063 0 2.72039063 0 0.083671875 12.5322363 0.083671875 16.1304785 6.08097656">

</polygon>
            <path d="M20.982832,7.25739258 C19.8342773,5.87425781 18.0166113,5.17297852 15.5803711,5.17297852 C13.4932324,5.17297852 10.9705078,6.17405273 8.0821582,8.14842773 C6.01330078,9.56267578 4.51705078,10.9627734 4.45438477,11.0216602 L3.43212891,11.9823047 L4.45438477,12.9429492 C4.51705078,13.0018359 6.01330078,14.4020215 8.0821582,15.8161816 C10.9705078,17.7905566 13.4932324,18.7916309 15.5803711,18.7916309 C18.0166113,18.7916309 19.8341895,18.0903516 20.982832,16.7072168 C21.929502,15.5671875 22.3897852,14.0216309 22.3897852,11.9822168 C22.3897852,9.94280273 21.9295898,8.39742187 20.982832,7.25739258 Z" id="Shape" fill="#666666">

</path>
            <rect id="Rectangle-path" fill="#000000" x="13.124707" y="10.2832031" width="2.63671875" height="2.63671875">

</rect>
            <polygon id="Shape" fill="#000000" points="41.6100586 34.3694531 34.6384863 16.774541 30.3499512 22.5569531 32.3580762 24.2755664 33.9329883 22.152041 39.818584 37.0061719 45 37.0061719 45 34.3694531">

</polygon>
        </g>
    </g>
</svg>
```

As you can see, SVGs can be large sometimes! Just to make it easier on us, lets make it a file and import it. 

- make an assets folder the same level as your /components and /pages folders

- inside of the assets folder, make a file called `error-handling-bug.svg` and place the svg code above into it **You do NOT need to export svg files! woo!**

- go back to Error.js and import the file

`import BugSVG from '../error-handling-bug.svg`

If you are wanting to use a .png or another image format, this process is the same.

```jsx
import BugSVG from '../error-handling-bug.svg'
import { Content } from 'react-bulma-components'

const Error = () => {
    return(
        <Content textSize={4} textAlign={'center'}>
            <h1>Oops! Our Bad!</h1>
            <img src={BugSVG} alt="bug error svg image" width={'500px'}/>
        </Content>
    )
}

export default Error
```

- give the error page a route to render on: 

Main.js

```jsx
import Error from "../pages/Error"

    [...]

  return (
    <main>
      <Routes>
        <Route path="/" element={<Home people={people} createPeople={createPeople}/>}/>
        <Route path="/error" element={<Error />}/>
        <Route path="/people" element={<Index people={people} />}/>
        <Route path="/people/:id" element={<Show people={people} updatePeople={updatePeople} deletePeople={deletePeople}/>}/>
      </Routes>
    </main>
  );
```

- go to `/error` to check it out!

## Lifting State

- what if you need data at different levels of your component hierarchy, but you dont need those props on the components between the beginning and where you need it? 

- there's a term for passing props through components that are a few levels or more deep: `Prop-Drilling`

- this is messy! its a pain! we can do better as devs! we can ...useContext()! 

- we need to import `createContext` from react in order to make our state context

-afterwards, lets get rid of all of our `people` props! 

Main.js

```jsx
//import createContext
import { useEffect, useState, createContext } from "react";

    [...]
    // removed people={people}
    <Routes>
        <Route path="/" element={<Home createPeople={createPeople}/>}/>
        <Route path="/error" element={<Error />}/>
        <Route path="/people" element={<Index />}/>
        <Route path="/people/:id" element={<Show updatePeople={updatePeople} deletePeople={deletePeople}/>}/>
    </Routes>
```

Removing our props.people form our Index and Show will break them, so don't worry! We'll fix that soon.

- lets create our state context "component", then wrap our routes in it

-wrapping our routes in our context component makes that context available to the entire app! woo!

Main.js

```jsx
[...]
import Error from "../pages/Error";

//we can use PeopleContext like a component, but it will be a component wrapped around the parent of all the child components that will need this context/state
export const PeopleContext = createContext()
//this exports our context so we can import it anywhere if we wish
const Main = (props) => {
[...]
```

- wrap our routes in our new context component called `PeopleContext`

Main.js

```jsx
      {/* <PeopleContext> is our context and .Provider is a built in function that we use to PROVIDE a value that will make the value usable by all components/routes */}
      <PeopleContext.Provider value={people}>
        <Routes>
          <Route path="/" element={<Home createPeople={createPeople}/>}/>
          <Route path="/error" element={<Error />}/>
          <Route path="/people" element={<Index />}/>
          <Route path="/people/:id" element={<Show updatePeople={updatePeople} deletePeople={deletePeople}/>}/>
        </Routes>
      </PeopleContext.Provider>
```

- now we need to import and assign the context value as needed. in this case its our Show.js, Home.js, and Index.js since they BOTH require access to props.people

Index.js / Home.js / Index.js

```jsx
import { PeopleContext } from '../components/Main'

const Index = (props) => {
    //props.people now becomes just 'people'!
    //what value do we assign to people? the one we passed into PeopleContext in Main.js
    const people = useContext(PeopleContext)
    //const state = useContext(CreateContextComponent)
```

##Voilà!##

## Can you change state with useContext()? 

Yes!!!! 

We won't be neeeding it in this example applicaiton, but here's how to start it. We'll leave the implimentation research/work to you! 

Main.js

```jsx
    //code above
    const [people, setPeople] = useState(null);
    //code between
    <PeopleContext.Provider value={{people, setPeople}}>
```

- lets see how we can easily use context to set our state as needed on any component!

Home.js | Show.js | Index.js

```jsx
    //place this where you normally put your state variables
    const [people, setPeople] = useContext(PeopleContext)
```

Yep! You can use it the same way we've always been using setState!

## Quick Hook Recap

- you have learned some common hooks now!

`useEffect, useState, useParams, useContext, useLocation` two of these hooks are from our good friend React Router which means...you can [make your own hooks too](https://react.dev/learn/reusing-logic-with-custom-hooks#extracting-your-own-custom-hook-from-a-component)! 

- highly recommended individual study: `useRef` and `useReducer` from react


# Challenges

Hungry for More? Want some ideas? 

- Figure out how to make the forms a dynamic component

- Rework your backend to handle the heavy lifting for goals like "filters" or "searching" 

- Pick another CSS react library and restyle

- DRY this code out! What code is repeated that can be a component or a function?

- Add a second model to your db and work it into the front/back ends
