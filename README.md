CakePHP State Machine
=====================
[![Build Status](https://travis-ci.org/tsmsogn/cakephp-state-machine.svg?branch=master)](https://travis-ci.org/tsmsogn/cakephp-state-machine)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/tsmsogn/cakephp-state-machine/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/tsmsogn/cakephp-state-machine/?branch=master)

Documentation is not finished yet either. See the tests if you want to learn something, as all aspects of the state machine is tested there.

## What is a State Machine?
http://en.wikipedia.org/wiki/State_machine

## Installation
First you need to alter the tables of the models you want to use StateMachine:
```sql
ALTER TABLE `vehicle` ADD `state` VARCHAR(50) NULL;
ALTER TABLE `vehicle` ADD `previous_state` VARCHAR(50) NULL;
```

## Features
- Callbacks on states and transitions
- Custom methods may be added to your model
- `is($state)`, `can($transition)`, `on($transition, 'before|after', callback)` and `when($state, callback)` methods allows you to control the whole flow. `transition($transition)` is used to move between two states.
- Roles and rules
- Graphviz

### Callbacks
You can add callbacks that will fire before/after a transition, and before/after a state change. This can either be done manually with `$this->on('mytransition', 'before', funtion() {})`, or you can add a method to your model:

```php
public function onBeforeTransition($currentState, $previousState, $transition) {
    // will fire on all transitions
}

public function onAfterIgnite($currentState, $previousState, $transition) {
    // will fire after the ignite transition
}
```

The state callbacks are a little different:

```php
public function onStateChange($newState) {
    // will fire on all state changes
}

public function onStateIdling($newState) {
    // will fire on the idling state
}
```


## Naming conventions
- Transitions and states in `$transitions` should be **lowercased** and **underscored**. The method names are in turn camelized.
  
  Example:

```
    shift_up   => canShiftUp() => shiftUp()
    first_gear => isFirstGear()
```

## How to Use
```php
App::uses('StateMachineBehavior', 'StateMachine.Model/Behavior');

class VehicleModel extends AppModel {

	public $useTable = 'Vehicle';

	public $actsAs = array('StateMachine.StateMachine');

	public $initialState = 'parked';

	public $transitionRules = array(
        'ignite' => array(
 			'role' => array('driver'),
			'depends' => 'has_key'
		)
	);

	public $transitions = array(
		'ignite' => array(
			'parked' => 'idling',
			'stalled' => 'stalled'
		),
		'park' => array(
			'idling' => 'parked',
			'first_gear' => 'parked'
		),
		'shift_up' => array(
			'idling' => 'first_gear',
			'first_gear' => 'second_gear',
			'second_gear' => 'third_gear'
		),
		'shift_down' => array(
			'first_gear' => 'idling',
			'second_gear' => 'first_gear',
			'third_gear' => 'second_gear'
		),
		'crash' => array(
			'first_gear' => 'stalled',
			'second_gear' => 'stalled',
			'third_gear' => 'stalled'
		),
		'repair' => array(
			'stalled' => 'parked'
		),
		'idle' => array(
			'first_gear' => 'idling'
		),
		'turn_off' => array(
			'all' => 'parked'
		)
	);

    public function __construct($id = false, $ds = false, $table = false) {
        parent::__construct($id, $ds, $table);
        $this->on('ignite', 'after', function($prevState, $nextState, $transition) {
            // the car just ignited!
        });
    }

    // a shortcut method for checking if the vehicle is moving
    public function isMoving() {
        return in_array($this->getCurrentState(), array('first_gear', 'second_gear', 'third_gear'));
    }

    // the dependant function for "ignite"
	public function hasKey($role) {
		return $role == 'driver';
	}

}
```

With the model above, we have the following methods:

```
isParked()    onStateParked()
isStalled()   onStateStalled()
ignite()      canIgnite()         onBeforeIgnite()    onAfterIgnite()
park()        canPark()           onBeforePark()      onAfterPark()
isFirstGear() onStateFirstGear()
shiftUp()     canShiftUp()        onBeforeShiftUp()   onAfterShiftUp()

....
```

```php
class Controller .... {
    public function method() {
        $this->Vehicle->create();
        $this->Vehicle->save(array(
            'Vehicle' => array(
                'title' => 'Toybota'
            )
        ));
        // $this->Vehicle->getCurrentState() == 'parked'
		if ($this->Vehicle->canIgnite(null, 'driver')) {
       	 	$this->Vehicle->ignite(null, 'driver');
       		$this->Vehicle->shiftUp();
        	// $this->Vehicle->getCurrentState() == 'first_gear'
		}
    }
}
```

## Graphviz
Here's how to state machine of the Vehicle would look like if you saved:
```php
$model->toDot()
```
into `fsm.gv` and ran:
```sh
dot -Tpng -ofsm.png fsm.gv
```
![](fsm.png)

## History

- Support PHP 5.3
- Add param $id to `is($state)`, `can($transition)` and `transition($transition)` methods. It specifies record to be read
