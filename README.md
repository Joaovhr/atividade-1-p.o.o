# atividade-1-p.o.o
#include <iostream>
#include <iomanip>
#include <stdexcept>

class Fan {
private:
    bool on;
    double cooling_power;

public:
    Fan(double power) : on(false), cooling_power(power) {
        if (cooling_power <= 0) {
            throw std::invalid_argument("A potencia de resfriamento deve ser positiva.");
        }
    }

    void turnOn() {
        on = true;
    }

    void turnOff() {
        on = false;
    }

    bool isOn() const {
        return on;
    }

    double coolingEffect() const {
        return on ? -cooling_power : 0.0;
    }
};

class TemperatureSensor {
private:
    double current_temp;
    double offset;

public:
    TemperatureSensor(double initial_temp, double offset_error)
        : current_temp(initial_temp), offset(offset_error) {}

    void drift(double temp_change) {
        current_temp += temp_change;
    }

    void clamp(double min_temp, double max_temp) {
        if (current_temp < min_temp) {
            current_temp = min_temp;
        }
        if (current_temp > max_temp) {
            current_temp = max_temp;
        }
    }

    double readC() const {
        return current_temp + offset;
    }
};

class OnOffController {
private:
    double turn_off_temp;
    double turn_on_temp;

public:
    OnOffController(double off_temp, double on_temp)
        : turn_off_temp(off_temp), turn_on_temp(on_temp) {
        if (turn_off_temp >= turn_on_temp) {
            throw std::invalid_argument("O limite para desligar deve ser menor que o para ligar.");
        }
    }

    void control(const TemperatureSensor& sensor, Fan& fan) {
        double current_temp = sensor.readC();

        if (current_temp > turn_on_temp) {
            fan.turnOn();
        } else if (current_temp < turn_off_temp) {
            fan.turnOff();
        }
    }
};

int main() {
    try {
        TemperatureSensor sensor(27.0, 0.0);
        Fan fan(0.4);
        OnOffController ctrl(24.0, 26.0);

        std::cout << std::fixed << std::setprecision(2);

        for (int t = 0; t < 30; ++t) {
            sensor.drift(+0.10);
            ctrl.control(sensor, fan);
            sensor.drift(fan.coolingEffect());
            sensor.clamp(0.0, 60.0);

            std::cout << "t=" << t << "s"
                      << " | T=" << sensor.readC() << " °C"
                      << " | Fan=" << (fan.isOn() ? "ON" : "OFF")
                      << "\n";
        }
    } catch (const std::exception& e) {
        std::cerr << "Erro: " << e.what() << "\n";
        return 1;
    }
    return 0;
}
