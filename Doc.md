# MMOS2324_Seminar2_Mateo_Trakostanec
---
## StepApp

StepApp je jednostavna React Native aplikacija koja koristi pedometar kako bi pratila korake korisnika. Korisnicima omogućuje unos željenog broja koraka koje žele ostvariti. Nakon što korisnik ostvari zadani broj koraka, aplikacija prikazuje poruku čestitke.

### Instalacija expo enviromenta
Najprije je potrebno skinuti i instalirati [Node.JS](https://nodejs.org/en/).

Zatim otvaramo terminal i izvršimo sljedeću naredbu kako bismo instalirali Expo CLI globalno:
```bash
npm install -g expo-cli
```
Stvorimo novi Expo projekt pomoću sljedeće naredbe:
```bash
npx expo init StepApp
```
Promijenimo se u direktorij  projekta pomoću sljedeće naredbe:
```bash
cd ImeVašeAplikacije
```
Pokrenemo Expo razvojni poslužitelj sljedećom naredbom:
```bash
npx expo start
```
Tom naredbom instalira se osnovna Expo aplikacija, zajedno s Expo Developer alatima koji omogućuju pristup i QR kodom za povezivanje aplikacije s mobilnim uređajem.

Prije samog razvoja, potrebno je također instalirati senzorski i linking modul koji će nam kasnije poslužiti za profiliranje.
```bash
expo install expo-sensors
expo install expo-linking
```


### App.js  
```js
import React, { useState, useEffect, useRef } from 'react';
import { View, Text, StyleSheet, TextInput, AppState, Keyboard, Alert, SafeAreaView } from 'react-native';
import { Pedometer } from 'expo-sensors';
import * as Linking from 'expo-linking';

const StepApp = () => {
  const [isPedometerAvailable, setIsPedometerAvailable] = useState('checking');
  const [pastStepCount, setPastStepCount] = useState(0);
  const [currentStepCount, setCurrentStepCount] = useState(0);
  const [goalSteps, setGoalSteps] = useState(25); // Default goal is 5000 steps
  const appState = useRef(AppState.currentState);

  const subscribe = async () => {
    const isAvailable = await Pedometer.isAvailableAsync();
    setIsPedometerAvailable(String(isAvailable));

    if (isAvailable) {
      const end = new Date();  // Current date and time
      const start = new Date();  // Current date and time

      // Set start to midnight
      start.setHours(0, 0, 0, 0);

      const pastStepCountResult = await Pedometer.getStepCountAsync(start, end);
      if (pastStepCountResult) {
        setPastStepCount(pastStepCountResult.steps);
      }

      return Pedometer.watchStepCount(result => {
        setCurrentStepCount(result.steps);
      });
    }
  };
  const url = Linking.useURL();

  if (url) {
    const { hostname, path, queryParams } = Linking.parse(url);

    console.log(
      `Linked to app with hostname: ${hostname}, path: ${path} and data: ${JSON.stringify(
        queryParams
      )}`
    );
  }

  const handleGoalChange = value => {
    const numericValue = parseInt(value, 10);
    setGoalSteps(isNaN(numericValue) ? 0 : numericValue);
  };

  const remainingSteps = goalSteps - currentStepCount;

  const dismissKeyboard = () => {
    Keyboard.dismiss();
  };

  const handleGoalCompletion = () => {
    Alert.alert(
      'Congratulations!',
      'You reached your goal!',
      [
        {
          text: 'Okay',
          onPress: () => {
            setGoalSteps(5000);
            setCurrentStepCount(0);
            console.log('Okay Pressed');
          },
        },
      ],
      { cancelable: false }
    );
  };

  const handleAppStateChange = nextAppState => {
    if (appState.current.match(/inactive|background/) && nextAppState === 'active') {
      subscribe();
    }
    appState.current = nextAppState;
  };

  useEffect(() => {
    AppState.addEventListener('change', handleAppStateChange);

    Pedometer.requestPermissionsAsync()
      .then(response => {
        if (response) {
          const subscription = subscribe();
          return () => subscription && subscription.remove();
        } else {
          console.log('Permission denied');
          setIsPedometerAvailable('not available');
        }
      })
      .catch(error => {
        console.error('Error requesting permissions:', error);
        setIsPedometerAvailable('not available');
      });

    return () => {
      Pedometer.removeEventListener('change', handleAppStateChange);
    };
  }, []);

  useEffect(() => {
    if (remainingSteps <= 0) {
      handleGoalCompletion();
    }
  }, [remainingSteps]);
  
  

  return (
    <SafeAreaView style={styles.safeArea}>
      <View style={styles.container}>
        <View style={styles.headerContainer}>
          <Text style={styles.header}>StepApp</Text>
        </View>

        {isPedometerAvailable !== 'checking' && isPedometerAvailable !== 'true' && (
          <Text style={styles.errorText}>Pedometer is not available on this device.</Text>
        )}

        <View style={styles.contentContainer} onTouchStart={dismissKeyboard}>
          <Text style={styles.text}>Steps taken today: {pastStepCount}</Text>
          <Text style={styles.text}>Walk! And watch this go up: {currentStepCount}</Text>

          <View style={styles.inputContainer}>
            <Text style={styles.inputLabel}>Set Your Goal:</Text>
            <TextInput
              style={styles.input}
              keyboardType="numeric"
              placeholder="Enter steps goal"
              value={goalSteps.toString()}
              onChangeText={handleGoalChange}
              returnKeyType="done"
              onSubmitEditing={dismissKeyboard}
            />
            
          </View>

          <Text style={styles.remainingText}>
            {remainingSteps > 0
              ? `You need to walk ${remainingSteps} more steps to reach your goal!`
              : 'Congratulations! You reached your goal!'}
          </Text>
        </View>
      </View>
    </SafeAreaView>
  );
};

const styles = StyleSheet.create({
  safeArea: {
    flex: 1,
    backgroundColor: '#f2f2f2', // Light gray background for SafeAreaView
  },
  container: {
    flex: 1,
    backgroundColor: '#f2f2f2', // Light gray background
  },
  headerContainer: {
    backgroundColor: '#333', // Dark gray header background
    paddingVertical: 20,
    alignItems: 'center',
  },
  header: {
    fontSize: 24,
    fontWeight: 'bold',
    color: 'white',
  },
  contentContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    paddingHorizontal: 20,
  },
  text: {
    fontSize: 18,
    marginBottom: 10,
    color: '#333', // Dark gray text
  },
  inputContainer: {
    marginBottom: 20,
    width: '100%',
  },
  inputLabel: {
    fontSize: 18,
    color: '#333', // Dark gray text
  },
  input: {
    height: 40,
    borderColor: '#666', // Medium gray border
    borderWidth: 1,
    borderRadius: 8,
    padding: 10,
    marginTop: 5,
    marginBottom: 10,
    backgroundColor: 'white',
  },
  remainingText: {
    fontSize: 18,
    color: '#333', // Dark gray text
    textAlign: 'center',
    marginTop: 20,
  },
  errorText: {
    fontSize: 16,
    color: 'red',
    marginBottom: 10,
    textAlign: 'center',
  },
});

export default StepApp;
```

### Izgled aplikacije 
