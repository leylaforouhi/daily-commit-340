import json
from pathlib import Path
from datetime import date

DATA_FILE = Path("workouts.json")


def load_workouts():
    if not DATA_FILE.exists():
        return []

    try:
        data = json.loads(DATA_FILE.read_text(encoding="utf-8"))
        return data if isinstance(data, list) else []
    except (json.JSONDecodeError, OSError):
        return []


def save_workouts(workouts):
    DATA_FILE.write_text(
        json.dumps(workouts, indent=2),
        encoding="utf-8",
    )


def add_workout(workouts, exercise, duration_minutes, calories=0):
    if not exercise.strip():
        raise ValueError("Exercise name cannot be empty")

    if duration_minutes <= 0:
        raise ValueError("Duration must be positive")

    if calories < 0:
        raise ValueError("Calories cannot be negative")

    workout = {
        "date": date.today().isoformat(),
        "exercise": exercise.strip(),
        "duration_minutes": duration_minutes,
        "calories": calories,
    }

    workouts.append(workout)
    save_workouts(workouts)
    return workout


def calculate_statistics(workouts):
    total_minutes = sum(
        item["duration_minutes"] for item in workouts
    )
    total_calories = sum(item["calories"] for item in workouts)

    return {
        "workout_count": len(workouts),
        "total_minutes": total_minutes,
        "total_hours": round(total_minutes / 60, 2),
        "total_calories": total_calories,
    }


if __name__ == "__main__":
    workouts = load_workouts()

    try:
        exercise = input("Exercise name: ")
        duration = int(input("Duration in minutes: "))
        calories = int(input("Estimated calories burned: "))

        add_workout(workouts, exercise, duration, calories)

        print("\nWorkout saved successfully!")
        print("\nTraining Statistics:")

        for key, value in calculate_statistics(workouts).items():
            print(f"{key.replace('_', ' ').title()}: {value}")

    except ValueError as error:
        print(f"Error: {error}")
