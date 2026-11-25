# React Native Example: A Native Task List with Shared React Logic

This example builds a small task list for a React Native app. It uses familiar React state and components while rendering native UI through React Native primitives.

The screen demonstrates:

- A controlled TextInput.
- Pressable controls with accessible names.
- FlatList for a changing collection.
- Stable task identifiers.
- Pure task logic kept separate from platform UI.
- A layout written with React Native styles.

## 1. Create a project

For a new app, the React Native team recommends starting with a framework. Create an Expo project with the current starter command:

    npx create-expo-app@latest rn-task-list
    cd rn-task-list

In the generated Expo Router project, put the screen below in app/index.tsx and the domain module in src/task-domain.ts. If your chosen template has a different entry point, keep the screen component and mount it from that entry point.

Start the development server using the command shown by the generated project. Open it on an Android or iOS device or emulator. Project templates and local platform requirements can change, so follow the setup instructions generated for your selected framework version.

## 2. Add the platform-neutral task logic

Create src/task-domain.ts:

    export type Task = {
      id: string;
      title: string;
      done: boolean;
    };

    export function normalizeTaskTitle(value: string): string {
      return value.trim();
    }

    export function toggleTask(tasks: Task[], targetId: string): Task[] {
      return tasks.map(task =>
        task.id === targetId
          ? { ...task, done: !task.done }
          : task,
      );
    }

    export function removeTask(tasks: Task[], targetId: string): Task[] {
      return tasks.filter(task => task.id !== targetId);
    }

These functions do not depend on React Native, the browser DOM, or a device API. A web screen could reuse this domain module while rendering its own suitable controls.

## 3. Build the native screen

Create app/index.tsx:

    import { useRef, useState } from 'react';
    import {
      FlatList,
      Pressable,
      StyleSheet,
      Text,
      TextInput,
      View,
    } from 'react-native';
    import {
      normalizeTaskTitle,
      removeTask,
      Task,
      toggleTask,
    } from '../src/task-domain';

    const starterTasks: Task[] = [
      { id: '1', title: 'Learn the React Native components', done: true },
      { id: '2', title: 'Build a controlled text input', done: false },
      { id: '3', title: 'Test the screen on both platforms', done: false },
    ];

    export default function Index() {
      const [tasks, setTasks] = useState(starterTasks);
      const [draft, setDraft] = useState('');
      const nextId = useRef(4);

      function addTask() {
        const title = normalizeTaskTitle(draft);

        if (title.length === 0) {
          return;
        }

        const newTask: Task = {
          id: String(nextId.current++),
          title,
          done: false,
        };

        setTasks(currentTasks => [newTask, ...currentTasks]);
        setDraft('');
      }

      function handleToggle(targetId: string) {
        setTasks(currentTasks => toggleTask(currentTasks, targetId));
      }

      function handleRemove(targetId: string) {
        setTasks(currentTasks => removeTask(currentTasks, targetId));
      }

      const remainingCount = tasks.filter(task => !task.done).length;

      return (
        <View style={styles.screen}>
          <Text style={styles.eyebrow}>React Native example</Text>
          <Text style={styles.title}>My tasks</Text>
          <Text style={styles.subtitle}>
            {remainingCount} remaining
          </Text>

          <View style={styles.entryRow}>
            <TextInput
              accessibilityLabel="New task"
              autoCorrect
              onChangeText={setDraft}
              onSubmitEditing={addTask}
              placeholder="Add a task"
              placeholderTextColor="#6b7280"
              returnKeyType="done"
              style={styles.input}
              value={draft}
            />

            <Pressable
              accessibilityLabel="Add task"
              accessibilityRole="button"
              disabled={!draft.trim()}
              onPress={addTask}
              style={({ pressed }) => [
                styles.addButton,
                !draft.trim() && styles.disabledButton,
                pressed && styles.pressed,
              ]}
            >
              <Text style={styles.addButtonText}>Add</Text>
            </Pressable>
          </View>

          <FlatList
            contentContainerStyle={
              tasks.length === 0 ? styles.emptyList : styles.list
            }
            data={tasks}
            keyExtractor={task => task.id}
            ListEmptyComponent={
              <Text style={styles.emptyMessage}>
                No tasks yet. Add one above.
              </Text>
            }
            renderItem={({ item }) => (
              <View style={styles.taskRow}>
                <Pressable
                  accessibilityLabel={
                    item.done
                      ? 'Reopen task ' + item.title
                      : 'Complete task ' + item.title
                  }
                  accessibilityRole="button"
                  onPress={() => handleToggle(item.id)}
                  style={({ pressed }) => [
                    styles.taskMain,
                    pressed && styles.pressed,
                  ]}
                >
                  <Text style={styles.taskMark}>
                    {item.done ? '✓' : '○'}
                  </Text>
                  <Text
                    style={[
                      styles.taskTitle,
                      item.done && styles.completedTitle,
                    ]}
                  >
                    {item.title}
                  </Text>
                </Pressable>

                <Pressable
                  accessibilityLabel={'Delete task ' + item.title}
                  accessibilityRole="button"
                  onPress={() => handleRemove(item.id)}
                  style={({ pressed }) => [
                    styles.deleteButton,
                    pressed && styles.pressed,
                  ]}
                >
                  <Text style={styles.deleteText}>Delete</Text>
                </Pressable>
              </View>
            )}
          />
        </View>
      );
    }

    const styles = StyleSheet.create({
      screen: {
        flex: 1,
        backgroundColor: '#f5f7fb',
        paddingHorizontal: 20,
        paddingTop: 32,
      },
      eyebrow: {
        color: '#52627a',
        fontSize: 13,
        fontWeight: '700',
        letterSpacing: 0.6,
        textTransform: 'uppercase',
      },
      title: {
        color: '#172033',
        fontSize: 30,
        fontWeight: '700',
        marginTop: 8,
      },
      subtitle: {
        color: '#52627a',
        fontSize: 16,
        marginTop: 6,
      },
      entryRow: {
        alignItems: 'center',
        flexDirection: 'row',
        gap: 10,
        marginTop: 24,
      },
      input: {
        backgroundColor: '#ffffff',
        borderColor: '#cbd3df',
        borderRadius: 10,
        borderWidth: 1,
        color: '#172033',
        flex: 1,
        fontSize: 16,
        minHeight: 48,
        paddingHorizontal: 14,
      },
      addButton: {
        alignItems: 'center',
        backgroundColor: '#2457d6',
        borderRadius: 10,
        justifyContent: 'center',
        minHeight: 48,
        minWidth: 68,
        paddingHorizontal: 16,
      },
      addButtonText: {
        color: '#ffffff',
        fontSize: 16,
        fontWeight: '700',
      },
      disabledButton: {
        opacity: 0.45,
      },
      pressed: {
        opacity: 0.72,
      },
      list: {
        gap: 10,
        paddingBottom: 28,
        paddingTop: 20,
      },
      emptyList: {
        flexGrow: 1,
        justifyContent: 'center',
        paddingVertical: 28,
      },
      emptyMessage: {
        color: '#52627a',
        fontSize: 16,
        textAlign: 'center',
      },
      taskRow: {
        alignItems: 'center',
        backgroundColor: '#ffffff',
        borderColor: '#d9e0ea',
        borderRadius: 12,
        borderWidth: 1,
        flexDirection: 'row',
        minHeight: 62,
        paddingHorizontal: 12,
      },
      taskMain: {
        alignItems: 'center',
        flex: 1,
        flexDirection: 'row',
        minHeight: 60,
        paddingVertical: 10,
      },
      taskMark: {
        color: '#2457d6',
        fontSize: 20,
        fontWeight: '700',
        marginRight: 12,
        width: 24,
      },
      taskTitle: {
        color: '#172033',
        flex: 1,
        fontSize: 16,
      },
      completedTitle: {
        color: '#697586',
        textDecorationLine: 'line-through',
      },
      deleteButton: {
        justifyContent: 'center',
        minHeight: 44,
        paddingHorizontal: 10,
      },
      deleteText: {
        color: '#a32735',
        fontSize: 14,
        fontWeight: '600',
      },
    });

## 4. How the example works

- The screen uses useState for the task collection and the input draft.
- The input is controlled: its value comes from draft and onChangeText updates that state.
- Adding a task trims its title, creates a stable identifier, and uses a functional state update.
- The task domain functions return new arrays and objects rather than mutating the existing state.
- FlatList receives a stable key from each task's id.
- Pressable provides the touch interaction and a meaningful accessibility label for each action.
- Styles are JavaScript objects. The layout uses React Native Flexbox rather than a browser stylesheet.
- The screen uses native components, while normalization and task updates are platform-neutral.

## 5. Try these changes

1. Add an edit action and update the task title without mutating the existing task.
2. Add a filter for all, remaining, and completed tasks.
3. Add a separate confirmation step before deleting a task.
4. Move the starter data into a reusable domain module.
5. Make a web version that reuses task-domain.ts but renders HTML controls.
6. Check the list with VoiceOver and TalkBack and improve labels if an action is unclear.

## 6. Verification checklist

- Add a task with the keyboard action and with the Add control.
- Confirm whitespace-only input does not create an item.
- Complete, reopen, and delete tasks.
- Confirm the empty state appears after removing every task.
- Check text wrapping and larger system text.
- Check keyboard appearance and dismissal.
- Test on iOS and Android if both are supported.
- Test accessibility with each platform's screen reader.

## Key lesson

React state and component composition work as expected, but the host components, events, styles, and accessibility behavior are native. Share pure task logic; render and verify the screen using the platform's components and conventions.
