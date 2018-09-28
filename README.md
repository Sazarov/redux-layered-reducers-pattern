# Redux sibling state interaction
A method for sibling state communication and interaction in Redux.

#### How it works
Say you have two or more reducers handling different parts of your state and you need these parts to interact with each other. In this example I have 2 sub-states that are on the same level:

````
state = {
        sliceA: {
            nameA: 'blue'
        },
        sliceB: {
            nameB: 'orange'
        }
};
````

We can change the values of the names but at some point we want to replace the value of `nameB` with the value in `nameA`. The state slices are handled by separate reducers -
`sliceA` is handled by reducer `reducerA` and `sliceB` - by `reducerB` like so:

````
function reducerA(state = {}, action) {
	switch ( action.type ) {
	
		case 'CHANGE_NAME_A': {
			return {
				nameA: action.data
			};
		}
		
		default:
			return state;
	}
};

function reducerB(state = {}, action) {
	switch ( action.type ) {
	
		case 'CHANGE_NAME_B': {
			return {
				nameB: action.data
			};
		}
		
		default:
			return state;
	}
};
````

In this case we wrap these reducers with another reducer that handles the interaction between them.

````
const nestedReducers = (state, action) => {
	return {
		sliceA: reducerA(state.sliceA, action),
		sliceB: reducerB(state.sliceB, action)
	}
};

const rootReducer = (state = {}, action) => {

	switch (action.type) {

		case 'REPLACE_B_WITH_A': {
			return {
				...state,
				sliceB: {
					nameB: state.sliceA.nameA
				}
			}
		}

		default: {
			const rest = nestedReducers(state, action);
			return {...state, ...rest}
		}

	}
};
````

`rootReducer` is our root reducer and has access to the entire state in this case. As such it can handle interaction between arbitrarily nested substates. What happens here is that when
an action of type `REPLACE_B_WITH_A` is dispatched, the wrapper reducer handles it by assigning the value currently in `nameA` to `nameB` in `sliceB`. All other action types are handled by the default
case in the switch statement. The `rest` object contains the all state slices that are wrapped by the wrapping reducer. If the dispatched action is not handled by the `rootReducer` it is pushed down
to the nested reducers.

The real benefit of this method comes when the state is deeply nested. Reducers which handle sibling state interactions need to be on the level of the interacting substates to have access to their 
parts of the state. This scales well because you don't handle all interactions in the root reducer but instead each set of substate interactions has their own reducer.

#### But why?
Sometimes reducerA needs information from reducerB to handle a particular action. The official redux documentation has excellent examples on sharing data between reducers and the method above is 
merely an alternative. It is certainly possible to send actions that are handled in by a reducer that handles a different part of the state with the necessary data as a payload, 
however that would result in spaghetti code. This method does not rely on comebineReducers to produce a nested state, while allowing your reducers to be granular and avoiding 
the "one huge reducer" problem.

#### Further reading

[Alternative methods for state sharing from the official documentation](https://redux.js.org/recipes/structuringreducers/beyondcombinereducers)

[combineReducers and reduceReducers explained](https://stackoverflow.com/questions/38652789/correct-usage-of-reduce-reducers/44371190#44371190)