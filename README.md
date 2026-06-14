# expense_tracker.py
```python
import json
import csv
import re
import time

from dataclasses import dataclass, asdict
from datetime import datetime, timedelta
from collections import defaultdict
from functools import wraps


# ==========================
# DATACLASS
# ==========================

@dataclass
class Xarajat:
    id: int
    tavsif: str
    summa: float
    kategoriya: str
    sana: str
    teglar: list


# ==========================
# DECORATOR
# ==========================

def timed(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()

        result = func(*args, **kwargs)

        finish = time.perf_counter()

        print(
            f"\n⏱ {func.__name__}: "
            f"{finish-start:.4f} sec"
        )

        return result

    return wrapper


# ==========================
# TRACKER
# ==========================

class Tracker:

    BUDGET = {
        "ovqat": 1000000,
        "transport": 500000,
        "oyin": 300000,
        "boshqa": 500000
    }

    def __init__(self):
        self.xarajatlar = []

    def __len__(self):
        return len(self.xarajatlar)

    def __iter__(self):
        return iter(self.xarajatlar)

    def __getitem__(self, index):
        return self.xarajatlar[index]

    def __repr__(self):
        return (
            f"Tracker("
            f"{len(self.xarajatlar)} xarajat)"
        )

    # ======================
    # PROPERTY
    # ======================

    @property
    def jami_summa(self) -> float:
        return sum(
            x.summa
            for x in self.xarajatlar
        )

    @property
    def o_rta_summa(self) -> float:

        if not self.xarajatlar:
            return 0

        return (
            self.jami_summa /
            len(self.xarajatlar)
        )

    @property
    def kategoriya_bo_yicha(self):

        result = defaultdict(float)

        for x in self.xarajatlar:
            result[x.kategoriya] += x.summa

        return dict(result)

    # ======================
    # REGEX
    # ======================

    def extract_tags(
        self,
        text: str
    ) -> list:

        return re.findall(
            r"#(\w+)",
            text
        )

    # ======================
    # ADD EXPENSE
    # ======================

    def add_expense(
        self,
        tavsif: str,
        summa: float
    ) -> None:

        tags = self.extract_tags(
            tavsif
        )

        kategoriya = (
            tags[0]
            if tags
            else "boshqa"
        )

        expense = Xarajat(
            id=len(self.xarajatlar)+1,
            tavsif=tavsif,
            summa=summa,
            kategoriya=kategoriya,
            sana=datetime.now().strftime(
                "%Y-%m-%d"
            ),
            teglar=tags
        )

        self.xarajatlar.append(
            expense
        )

    # ======================
    # LIST
    # ======================

    def list_expenses(
        self
    ) -> None:

        if not self.xarajatlar:
            print(
                "\nXarajatlar yo'q"
            )
            return

        for x in self.xarajatlar:

            print(
                f"[{x.id}] "
                f"{x.tavsif} | "
                f"{x.summa:,.0f} so'm | "
                f"{x.kategoriya} | "
                f"{x.sana}"
            )

    # ======================
    # FILTER
    # ======================

    def filter_category(
        self,
        category: str
    ):

        return [
            x
            for x in self.xarajatlar
            if x.kategoriya.lower()
            == category.lower()
        ]

    # ======================
    # TOP CATEGORY
    # ======================

    def top_categories(self):

        return sorted(
            self.kategoriya_bo_yicha.items(),
            key=lambda x: x[1],
            reverse=True
        )

    # ======================
    # LAST 7 DAYS
    # ======================

    def last_7_days(self):

        now = datetime.now()

        return [
            x
            for x in self.xarajatlar
            if (
                now -
                datetime.strptime(
                    x.sana,
                    "%Y-%m-%d"
                )
            ).days <= 7
        ]

    # ======================
    # BUDGET CHECK
    # ======================

    def check_budget(self):

        for cat, total in (
            self.kategoriya_bo_yicha.items()
        ):

            if (
                cat in self.BUDGET
                and
                total > self.BUDGET[cat]
            ):

                print(
                    f"⚠ {cat} "
                    f"limiti oshdi!"
                )

    # ======================
    # SAVE JSON
    # ======================

    def save(
        self,
        filename="expenses.json"
    ) -> None:

        with open(
            filename,
            "w",
            encoding="utf-8"
        ) as f:

            json.dump(
                [
                    asdict(x)
                    for x in self.xarajatlar
                ],
                f,
                ensure_ascii=False,
                indent=4
            )

        print(
            "\nSaqlandi!"
        )

    # ======================
    # LOAD JSON
    # ======================

    def load(
        self,
        filename="expenses.json"
    ) -> None:

        try:

            with open(
                filename,
                "r",
                encoding="utf-8"
            ) as f:

                data = json.load(f)

            self.xarajatlar = [
                Xarajat(**item)
                for item in data
            ]

            print(
                "\nYuklandi!"
            )

        except FileNotFoundError:

            print(
                "\nFayl topilmadi"
            )

    # ======================
    # CSV EXPORT
    # ======================

    def export_csv(
        self,
        filename="expenses.csv"
    ):

        with open(
            filename,
            "w",
            newline="",
            encoding="utf-8-sig"
        ) as f:

            writer = csv.DictWriter(
                f,
                fieldnames=[
                    "id",
                    "tavsif",
                    "summa",
                    "kategoriya",
                    "sana",
                    "teglar"
                ]
            )

            writer.writeheader()

            for x in self.xarajatlar:
                writer.writerow(
                    asdict(x)
                )

        print(
            "\nCSV export qilindi!"
        )

    # ======================
    # REPORT
    # ======================

    @timed
    def report(self):

        print("\n===== HISOBOT =====")

        print(
            f"Jami xarajat: "
            f"{self.jami_summa:,.0f}"
        )

        print(
            f"O'rtacha: "
            f"{self.o_rta_summa:,.0f}"
        )

        print(
            f"Jami soni: "
            f"{len(self)}"
        )

        print(
            "\nTOP kategoriyalar:"
        )

        for cat, total in (
            self.top_categories()
        ):
            print(
                f"{cat}: "
                f"{total:,.0f}"
            )

        self.check_budget()


# ==========================
# CLI
# ==========================

def main():

    tracker = Tracker()

    while True:

        print("""
======== MENU ========

1. Xarajat qo'shish
2. Xarajatlar ro'yxati
3. Statistika
4. Kategoriya filtri
5. Saqlash
6. Yuklash
7. CSV export
8. Oxirgi 7 kun

0. Chiqish

======================
""")

        choice = input(
            "Tanlang: "
        )

        if choice == "1":

            desc = input(
                "Tavsif: "
            )

            amount = float(
                input(
                    "Summa: "
                )
            )

            tracker.add_expense(
                desc,
                amount
            )

            print(
                "Qo'shildi!"
            )

        elif choice == "2":

            tracker.list_expenses()

        elif choice == "3":

            tracker.report()

        elif choice == "4":

            cat = input(
                "Kategoriya: "
            )

            result = (
                tracker.filter_category(
                    cat
                )
            )

            for x in result:
                print(x)

        elif choice == "5":

            tracker.save()

        elif choice == "6":

            tracker.load()

        elif choice == "7":

            tracker.export_csv()

        elif choice == "8":

            for x in (
                tracker.last_7_days()
            ):
                print(x)

        elif choice == "0":

            print(
                "Dastur tugadi"
            )
            break

        else:

            print(
                "Noto'g'ri tanlov!"
            )


if __name__ == "__main__":
    main()
```
