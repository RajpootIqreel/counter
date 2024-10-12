# counter
import React, { useEffect, useState } from 'react';
import {
  StyleSheet,
  Text,
  View,
  Button,
  Alert,
} from 'react-native';
import { Pedometer } from 'react-native-pedometer'; // Use 'expo-sensors' if using Expo
import AsyncStorage from '@react-native-async-storage/async-storage';
import PushNotification from 'react-native-push-notification';

export default function App() {
  const [steps, setSteps] = useState(0);
  const [dailySteps, setDailySteps] = useState(0);
  const stepGoal = 10000;

  useEffect(() => {
    const subscribeToPedometer = async () => {
      const isPedometerAvailable = await Pedometer.isStepCountingAvailableAsync();
      if (isPedometerAvailable) {
        Pedometer.watchStepCount((result) => {
          setSteps(result.steps);
          handleGoal(result.steps);
        });
      } else {
        Alert.alert("Pedometer not available on this device");
      }
    };

    const resetStepsAtMidnight = () => {
      const now = new Date();
      const nextMidnight = new Date();
      nextMidnight.setHours(24, 0, 0, 0);
      const timeUntilMidnight = nextMidnight - now;

      setTimeout(() => {
        resetDailySteps();
        resetStepsAtMidnight(); // Call again to reset next day
      }, timeUntilMidnight);
    };

    subscribeToPedometer();
    resetStepsAtMidnight();
  }, []);

  const handleGoal = (currentSteps) => {
    if (currentSteps >= stepGoal) {
      PushNotification.localNotification({
        title: "Goal Reached!",
        message: `You have reached your goal of ${stepGoal} steps!`,
      });
    }
  };

  const resetDailySteps = async () => {
    try {
      setDailySteps(steps);
      await AsyncStorage.setItem('dailySteps', steps.toString());
      setSteps(0); // Reset steps
    } catch (error) {
      console.log("Error resetting steps", error);
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Step Counter</Text>
      <Text style={styles.steps}>Current Steps: {steps}</Text>
      <Text style={styles.steps}>Today's Steps: {dailySteps}</Text>
      <Button title="Reset Daily Steps" onPress={resetDailySteps} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
  },
  title: {
    fontSize: 24,
    marginBottom: 20,
  },
  steps: {
    fontSize: 18,
    marginBottom: 10,
  },
});
