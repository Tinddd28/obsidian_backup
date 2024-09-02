1) Разобраться с инициализацией БД
2) Разобраться с локальным запуском для отладки
3) Модификация модели проекта, операций, пользователя для реализация транзакций - 1-2 дня

```
class Projects(Base):
    """Model for projects."""

    __tablename__ = "projects"

    id: Mapped[int] = mapped_column(
        BigInteger, unique=True, primary_key=True
    )
    title: Mapped[str] = mapped_column(String, unique=True)
    description: Mapped[str] = mapped_column(Text)
    token_title: Mapped[str] = mapped_column(String, unique=True)
    image: Mapped[str] = mapped_column(String, unique=True)
    amount: Mapped[int] = mapped_column(Integer)
    cost_per_token: Mapped[float] = mapped_column(Float, index=True)
    total_supply: Mapped[int] = mapped_column(Integer)
    locked_tokens: Mapped[int] = mapped_column(Integer, default=0)
    unlock_date: Mapped[datetime] = mapped_column(DateTime)
    price_per_token: Mapped[float] = mapped_column(Float)

    wallets: Mapped[list["Wallets"]] = relationship(
        "Wallets", back_populates="project", lazy="selectin"
    )
    operations: Mapped[list["Operations"]] = relationship(
        "Operations", back_populates="project", lazy="selectin"
    )
```
примерная реализация модели проекта, протестировать и исправить

```
class Operations(Base):
    """Model for operations."""

    __tablename__ = "operations"

    id: Mapped[int] = mapped_column(
        BigInteger, unique=True, primary_key=True
    )
    ticket_project: Mapped[int] = mapped_column(BigInteger, index=True)
    user_id: Mapped[int] = mapped_column(BigInteger, ForeignKey(
        "user.id", ondelete="CASCADE", onupdate="CASCADE"
    ))
    project_id: Mapped[int] = mapped_column(BigInteger, ForeignKey(
        "projects.id", ondelete="CASCADE", onupdate="CASCADE"
    ))
    quantity: Mapped[float] = mapped_column(Integer)
    total_amount: Mapped[float] = mapped_column(Float)
    tx_hash: Mapped[str] = mapped_column(
        String, unique=True, index=True
    )
    transaction_date: Mapped[datetime] = mapped_column(DateTime)
    is_locked: Mapped[bool] = mapped_column(Boolean, default=True)
    user: Mapped["User"] = relationship(
        "User", back_populates="operations"
    )
    project: Mapped["Projects"] = relationship(
        "Projects", back_populates="operations"
    )
```

Аналогично с операциями

```
balance = Column(Float, default=0.0)

locked_balance = Column(Float, default=0.0)

transactions = relationship('Transaction', back_populates='user')
```

модификация модели пользователя


дополнить логику для модели транзакций
```
class Transaction(Base):

__tablename__ = 'transactions'

id = Column(Integer, primary_key=True)

user_id = Column(Integer, ForeignKey('users.id'))

project_id = Column(Integer, ForeignKey('projects.id'))

quantity = Column(Float)

is_locked = Column(Boolean, default=True)

unlock_date = Column(DateTime)

user = relationship('User', back_populates='transactions')

project = relationship('Project', back_populates='transactions')
```

4) выяснить какие сети используются, тестировать на эфире, а точнее его сетях для теста, к примеру - меньше 1 дня
5) Функции для покупки (создание фабрики для работы с разными сетями) - 3 дня
```
def purchase_tokens(session: Session, user_id: int, project_id: int, quantity: float, price_per_token: float):
```
5) Продумать, как работать с разлоком - реализовать - 2 дня
6) Продумать/реализовать/узнать о миграции 
7) реализовать crud операции для работы покупки токенов и продаже - 2-3 дня
8) протестировать работу транзакций вне основного кода - в купе с предыдущим пунктом
9) создание эндпоинтов для обработки запросов 

_В идеале, сопроводить тестами следующие моменты:_ - 3 дня
- работу с БД: таблицы пользователей, сетей, транзакций
- работу с осуществлением транзакций