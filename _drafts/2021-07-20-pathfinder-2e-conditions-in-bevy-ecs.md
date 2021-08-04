---
published: false
---
## Pathfinder 2e Conditions in Bevy ECS



```rust
use bevy::prelude::*;

pub struct ConditionsPlugin;

/// This plugin is used for adding and removing conditions from entities.
/// Conditions need to mutate the creature(entity) once but then remain “on” the character. Events trigger adding and removing conditions
impl Plugin for ConditionsPlugin {
    fn build(&self, app: &mut AppBuilder) {
        app.add_event::<AddConditionEvent<Flatfooted>>()
            .add_event::<RemoveConditionEvent>()
            .add_system(add_flatfooted_event_reader.system())
            .add_system(flatfooted_condition_event_writer.system())
            .add_system(flatfooted.system())
            .add_system(remove_flatfooted_event_reader.system());
        // .add_system(next_turn.system());
    }
}

#[derive(Component, Debug, Clone)]
struct Condition<T: ConditionFunctions> {
    applied: bool,
    condition: T,
}

pub trait ConditionFunctions {
    fn add(&self, entity: Entity, commands: &mut Commands);
    fn remove(&self, entity: Entity, commands: &mut Commands);
}

/// Use this in a system to mark an entity that needs to be mutated by the condition.
struct NeedsCondition<T: ConditionFunctions> {
    condition: Condition<T>,
}

struct AddConditionEvent<T: ConditionFunctions> {
    condition: Condition<T>,
}
struct RemoveConditionEvent {
    entities: Vec<Entity>,
}

/// ----------------------------------------------
///         # Flatfooted Condition Code
/// ----------------------------------------------
#[derive(Debug, Clone, Copy)]
struct Flatfooted;

impl ConditionFunctions for Flatfooted {
    fn add(&self, entity: Entity, commands: &mut Commands) {
        eprintln!("Flatfooted condition added to {:?}", entity);
    }
    fn remove(&self, entity: Entity, commands: &mut Commands) {
        eprintln!("Flatfooted condition removed from {:?}", entity);
    }
}

/// # Flatfooted System
/// This system mutates any actor with the Flatfooted condition and removes the NeedsFlatFooted condition if still present
fn flatfooted(mut query: Query<(Entity, &NeedsCondition<Flatfooted>)>, mut commands: Commands) {
    for (entity, needs) in query.iter_mut() {
        let mut flatfooted = needs.condition.clone();
        if !flatfooted.applied {
            flatfooted.condition.add(entity, &mut commands);
            flatfooted.applied = true;
            commands
                .entity(entity)
                .remove::<NeedsCondition<Flatfooted>>()
                .insert(flatfooted);
            eprintln!("{:?} has been mutated to be flatfooted.", entity);
        }
    }
}

/// Event Writer system to trigger AddCondition<Flatfooted> (just keyboard for now). Just an example.
fn flatfooted_condition_event_writer(
    mut writer: EventWriter<AddConditionEvent<Flatfooted>>,
    mut writer2: EventWriter<RemoveConditionEvent>,
    query: Query<Entity, With<Condition<Flatfooted>>>,
    keys: Res<Input<KeyCode>>,
) {
    if keys.just_pressed(bevy::input::prelude::KeyCode::A) {
        let condition = Condition::<Flatfooted> {
            applied: false,
            condition: Flatfooted,
        };
        eprintln!("INPUT -- A was pressed. Firing AddConditionEvent with Flatfooted condition.");
        let condition_event = AddConditionEvent::<Flatfooted> { condition };
        writer.send(condition_event);
    }
    if keys.just_pressed(bevy::input::prelude::KeyCode::R) {
        eprintln!("INPUT -- R was pressed. Firing RemoveConditionEvent<Flatfooted>");
        let remove_event = RemoveConditionEvent {
            entities: query.iter().collect(),
        };
        eprint!("{:?} ----", remove_event.entities);
        writer2.send(remove_event);
    }
}

/// Reader just to mark the entity as needing the flatfooted condition. Exmple in this case, just adds to all entities.
fn add_flatfooted_event_reader(
    mut reader: EventReader<AddConditionEvent<Flatfooted>>,
    mut query: Query<(Entity, Without<Condition<Flatfooted>>)>,
    mut commands: Commands,
) {
    for event in reader.iter() {
        for (entity, _) in query.iter_mut() {
            eprintln!("Adding Condtion<Flatfooted> to entity: {:?}", entity);
            commands
                .entity(entity)
                .insert(NeedsCondition::<Flatfooted> {
                    condition: event.condition.clone(),
                });
        }
    }
}

fn remove_flatfooted_event_reader(
    mut reader: EventReader<RemoveConditionEvent>,
    query: Query<&Condition<Flatfooted>>,
    mut commands: Commands,
) {
    for event in reader.iter() {
        eprintln!(
            "{:?} entities need flatfooted removed",
            event.entities.len()
        );
        for &entity in event.entities.iter() {
            eprintln!("Removing flatfooted from entity: {:?}", entity);
            let component = query.get(entity);
            match component {
                Ok(component) => component.condition.remove(entity, &mut commands),
                Err(err) => eprintln!(
                    "Error removing flatfooted from entity: {:?}; {:?}",
                    entity, err
                ),
            };
            commands.entity(entity).remove::<Condition<Flatfooted>>();
        }
    }
}
```
