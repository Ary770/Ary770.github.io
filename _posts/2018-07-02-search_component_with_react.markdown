---
layout: post
title:      "Search Component With React "
date:       2018-07-02 17:27:59 -0400
permalink:  search_component_with_react
---


One of the features I incorporated in my [Herbal Remedies App](http://https://github.com/Ary770/herbal-remedies-app) is the ability to search for herbs by medicinal use and properties.

There are a few ways to accomplish this. 

The first idea that comes to mind would be to make a fetch request to the API where the data sits, and include search parameters for the desired resource. But this way is potentially slow, especially if we wanted to display the matching data as the user types.

So the route I took was to make a single call to the API, fetching all the data we need for the application and handle the search on the client side. 

The way the app is wired up is a bit complex (as most react applications are in my opinion) so bear with me.

First, we have the Herbs component, which is a simple, functional component that looks like this:

```
import React from 'react';
import { Link } from 'react-router-dom';

const Herbs = (props) => {
  let herbs = null;

  if (props.herbs) {
    herbs = props.herbs.map(herb =>
      <h4 key={herb.id}><li role='presentation'><Link to={`${props.url}/${herb.id}`}>{herb.name}</Link></li></h4>
    )
  }

  return (
    <div className="col-sm-3">
      <ul className="nav nav-pills nav-stacked">
        {herbs}
      </ul>
    </div>
  )
}

export default Herbs;
```

As you can see, the purpose of this component is to display a <Link/> element with a nested route, of all the herbs that are being passed as props to the component. 

Then we have the HerbShow container:

```
import React from 'react';
import { connect } from 'react-redux';
import PanelWrapper from '../components/PanelWrapper';
import '../App.css';

const HerbShow = ({ herb }) => {
  const panel = <PanelWrapper key={herb.id} herb={herb}/>

  return (
    <div className='col-md-8'>
      <div className='Static'>
        {panel}
      </div>
    </div>
  )
}

const mapStateToProps = (state, ownProps) => {
  const herb = state.herbs.herbs.find(herb =>
    herb.id.toString() === ownProps.match.params.herbId
  )

  if (herb) {
    return { herb }
  } else {
    return { herb: {} }
  }
}

export default connect(mapStateToProps)(HerbShow);
```

This container connects to the State (managed by Redux), finds the herb that matches the id by introspecting on the parameters of the route, the 'herbId', and creates a <PanelWrapper/> to display the data.

Once we have these two components in place, we can code our Search Container:

```
import React from 'react';
import { connect } from 'react-redux';
import Herbs from '../components/Herbs';
import HerbShow from './HerbShow';
import { Route } from 'react-router-dom';

class SearchByMedicinalUses extends React.Component {
  state = {
    showHerbs: null,
  }

  handleSearch = (event) => {
    const text = event.target.value
    if (text !== "") {
      const medicinalUse = text.split(' ').map(
        w => w.charAt(0).toUpperCase() + w.substr(1)
      ).join(' ');

      const matchingHerbs = this.props.herbs.filter(herb =>
        herb.medicinal_uses && herb.medicinal_uses.includes(medicinalUse.trim())
      );

      if (matchingHerbs) {
        this.setState({
          showHerbs: matchingHerbs
        })
      }
    } else {
      this.setState({ showHerbs: null })
      this.props.history.replace('/medicinal-uses')
    }
  }

  render() {
    let herbs = null;
    let results = this.state.showHerbs;

    if (results) {
      herbs = <Herbs url={this.props.match.url} herbs={results}/>
    }

    return (
      <div className="row">
        <h1>Search by Medicinal Uses</h1>
        <div className="col-lg-6">
          <input
            type="text"
            className="form-control"
            onChange={(event) => this.handleSearch(event)}
            placeholder="Search for..."
            />
          <br></br>
          {herbs}
          <Route path={`${this.props.match.url}/:herbId`} component={HerbShow}/>
        </div>
      </div>
    )
  }
}

const mapStateToProps = state => {
  return ({
    herbs: state.herbs.herbs
  })
}

export default connect(mapStateToProps)(SearchByMedicinalUses)
```

Ok, lots of code...

This component renders an input element that 'onChange', calls a function named handleSearch. So every time a user types into the input field, handleSearch is called. 

Let's break down this function.

1. Get the value of the input:

`const text = event.target.value`

2. Since I only want to display data if the user types something, we'll wrap the code into in a conditional statement: 

```
if (text !== "") {
	...do something
    } else {
	..do something else
 }
```

3. If the text is not empty, I need to format the text to match the data in my database. In this case, it means capitalizing the first letter of each word the user types:

```
const medicinalUse = text.split(' ').map(
 	w => w.charAt(0).toUpperCase() + w.substr(1)
).join(' ');
```

4. Now we are ready to do the actual search:

```
 const matchingHerbs = this.props.herbs.filter(herb =>
  	herb.medicinal_uses  //some fileds are empty so before we call includes I need to make sure the field exists
&& herb.medicinal_uses.includes(medicinalUse.trim())
      );
```

5. If there is a match, set the local state to the new array of matchingHerbs:

```
this.setState({
   	showHerbs: matchingHerbs
})
```

With this in place, if this.state.showHerbs is populated, we can conditionally render the <Herbs/> component and pass it the matching results through props:

```
let herbs = null;
let results = this.state.showHerbs;

if (results) {
herbs = <Herbs url={this.props.match.url} herbs={results}/>
}
```


And since <Herbs/> component renders a <Link/> for each herb we need to include a <Route/> element mapped to the <HerbShow/> component for our the links to work properly:

```
{herbs}
<Route path={`${this.props.match.url}/:herbId`} component={HerbShow}/>
```

Finally, if or when the text is deleted to perform a new search, I would like the data cleared, hence the else statement in our handleSearch function:

```
 else {
      this.setState({ showHerbs: null })
      this.props.history.replace('/medicinal-uses')
 }
```

With these two lines of code, we can 1) clear the <Herbs/> components that are being rendered conditionally and 2) clear the <ShowHerb/> component that is being displayed based on the route params.

