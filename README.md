from abc import ABC, abstractmethod
from datetime import date


# 1 Abstraction
class MenuItem(ABC):
    def __init__(self, name, base_price):
        self.name = name
        self.base_price = base_price

    @abstractmethod
    def calculate_price(self):
        pass

    def get_display_name(self):
        return self.name


# dish
class Pizza(MenuItem):
    def __init__(self, name, base_price, size):
        super().__init__(name, base_price)
        self.size = size  # "маленькая", "средняя", "большая"

    def calculate_price(self):
        if self.size == "большая":
            return self.base_price * 1.3
        return self.base_price

    def get_display_name(self):
        return f"{self.name} ({self.size})"


class Drink(MenuItem):
    def __init__(self, name, base_price, is_ice=False):
        super().__init__(name, base_price)
        self.is_ice = is_ice

    def calculate_price(self):
        return self.base_price  # лёд цену не меняет

    def get_display_name(self):
        return self.name + " со льдом" if self.is_ice else self.name


class ComboSet(MenuItem):
    def __init__(self, name, items):
        super().__init__(name, 0)
        self.items = items  # список других MenuItem

    def calculate_price(self):
        total = sum(item.calculate_price() for item in self.items)
        return total * 0.85  # скидка 15%

    def get_display_name(self):
        names = ", ".join(item.get_display_name() for item in self.items)
        return f"{self.name} [{names}]"


# payment
class PaymentStrategy(ABC):
    @abstractmethod
    def collect_payment(self, amount):
        pass


class CardPayment(PaymentStrategy):
    def __init__(self, card_number, exp_month, exp_year):
        self.card_number = card_number
        self.exp_month = exp_month
        self.exp_year = exp_year

    def _is_valid(self):
        today = date.today()
        return (self.exp_year, self.exp_month) >= (today.year, today.month)

    def collect_payment(self, amount):
        if not self._is_valid():
            print("Ошибка: срок действия карты истёк, оплата отклонена.")
            return False
        print(f"С карты **** {self.card_number[-4:]} списано {amount:.2f} сом.")
        return True


class CashPayment(PaymentStrategy):
    def collect_payment(self, amount):
        print(f"Оплата наличными: курьеру нужно отдать {amount:.2f} сом.")
        return True


# Incapsulation
class Order:
    STATUSES = ["Формируется", "Готовится", "Доставляется", "Доставлен"]

    def __init__(self):
        self.__items = []     # приватный список блюд
        self.__status = 0     # приватный индекс статуса

    def get_status(self):
        return self.STATUSES[self.__status]

    def next_step(self):
        if self.__status < len(self.STATUSES) - 1:
            self.__status += 1
        else:
            print("Заказ уже доставлен, дальше некуда.")

    def add_item(self, item):
        if self.__status >= 1:  # "Готовится" или дальше
            raise Exception("Нельзя добавить блюдо: заказ уже готовится!")
        self.__items.append(item)

    def get_total_cost(self):
        return sum(item.calculate_price() for item in self.__items)

    def pay(self, payment_method):
        return payment_method.collect_payment(self.get_total_cost())

    def show(self):
        print("--- Заказ ---")
        for item in self.__items:
            print(f"{item.get_display_name()} — {item.calculate_price():.2f}")
        print(f"Итого: {self.get_total_cost():.2f} | Статус: {self.get_status()}")


# demo
if __name__ == "__main__":
    pizza = Pizza("Маргарита", 500, "большая")
    cola = Drink("Кола", 100, is_ice=True)
    combo = ComboSet("Обед", [Pizza("Пепперони", 450, "средняя"), Drink("Чай", 80)])

    order = Order()
    order.add_item(pizza)
    order.add_item(cola)
    order.add_item(combo)
    order.show()

    order.pay(CardPayment("1234567812345678", 12, 2027))
    order.pay(CardPayment("1234567812345678", 1, 2020))  # просрочена
    order.pay(CashPayment())

    order.next_step()  # Готовится
    try:
        order.add_item(Drink("Сок", 120))
    except Exception as e:
        print("Ошибка:", e)

    order.next_step()
    order.next_step()
    print("Статус:", order.get_status())
