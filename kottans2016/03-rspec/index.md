# RSpec и подходы к тестированию на Ruby

**Kottans-Ruby**, осень 2016

Лекция **Третья** (не считая мастер-класса)

**Виктор Шепелев** (zverok)

# Что такое тестирование и зачем оно надо

Давайте построим самолёт!

!SLIDE

День первый...

{% coderay %}
# plane.rb
class Plane
  def fly!
  end
end

# test_plane.rb
plane = Plane.new

plane.fly!
{% end %}

!SLIDE

День 1015...

{% coderay %}
# test_plane.rb

# plane = Plane.new
# plane.fly! => EngineError: Not enough gasoline!

plane = Plane.new(gasoline: 3000)
# plane.fly! => PathError: You are not at runway!

plane.drive_to(Runway.first)
# plane.fly! => TechError: Ladder is still connected!

plane.ladders.each(&:disconnect!)
# plane.fly! => SpeedError: Not enough speed for flying!

plane.speedup(until: Plane::TAKEOFF_SPEED)
plane.fly!
# => CriticalError: Engine malfunction. 3 crew members and 74 passengers dead.
{% end %}

!SLIDE

ОК, тестируем только двигатель...

{% coderay %}
e = Engine.new
p e
p e.active?
p e.speed
e.activate!
p e.active?
p e.speed
{% end %}

<% code(lang: 'text') do %>
#<Plane::EnginesGasolineEngine:0x9db6240 ..................>
false
0
true
ZeroDivisionError: divided by 0
  ....1000 lines of backtrace...
<% end %>

## Инструменты тестирования: RSpec

{% coderay %}
describe Engine do
  subject { described_class.new }
  it { is_expected.to be_an Engine }

  context 'when not activated' do
    it { is_expected.not_to be_activated }
    its(:speed) { is_expected.to eq 0 }
  end

  context 'when activated' do
    before { subject.activate! }

    it { is_expected.to be_activated }
    its(:speed) { is_expected.to eq 1000 }
  end
end
{% end %}

## Базовый тест

* Expectation — ожидание:

{% coderay %}
  expect(engine.speed).to eq 1000
{% end %}

* Единичный тест:

{% coderay %}
it 'has some speed' do
  engine = Engine.new
  engine.start!
  expect(engine.speed).to eq 1000
end
{% end %}

## Начальные (и финальные) действия

{% coderay %}
before { engine.start }
after { engine.stop }

before(:all) { Laboratory.open!(password: 'Дядя Вася') }
{% end %}

## Объекты, участвующие в тесте (`let`)

* Без `let`:
{% coderay %}
before {
  @engine = Engine.new
  @engine.start
}
it 'has some speed' do
  expect(@engine.speed).to eq 1000
end
{% end %}

* С `let`:
{% coderay %}
let(:engine) { Engine.new }
before { engine.start }
it 'has some speed' do
  expect(engine.speed).to eq 1000
end
{% end %}

## Кого мы тестируем? (`subject`)

{% coderay %}
subject { Engine.new }
it 'has some speed' do
  expect(subject.speed).to eq 1000
end
{% end %}

... тоже может иметь имя:
{% coderay %}
subject(:engine) { Engine.new }
it 'has some speed' do
  expect(engine.speed).to eq 1000
end
{% end %}

## `subject` и краткость тестов

{% coderay %}
subject(:engine) { Engine.new }

it { is_expected.to be_activated }
it { is_expected.to have_attributes(speed: 1000) }

# с гемом rspec-its:
its(:speed) { is_expected.to eq 1000 }
{% end %}

## `let` и `let!`

* `let`: создаётся только при явном обращении:
{% coderay %}
let(:tank) { GasTank.new(100) }
subject { Engine.new(tank) }
{% end %}

* `let!`: создаётся всегда:
{% coderay %}
let!(:environment) { TestBench.create }
subject { Engine.new }
{% end %}

PS: `subject!` тоже существуует. И иногда полезен.

## Проверки (matchers)

{% coderay %}
expect(engine.speed).to eq 1000
expect(engine.speed).to be > 800
expect(engine.speed).to be_within(0.1).of(1000)

expect(engine).to be_an Engine

expect(engine).to be_activated # calls engine.activated?

expect(engine.components).to include('Carburator')
expect(engine.components).to contain_exactly('Carburator', 'Ignition', 'Cylinder', 'Cylinder')

{% end %}

## Съешь ещё этих мягких французских matchers

Используем блоки кода:

{% coderay %}
expect { engine.speed = 10_000 }.to raise_error(ArgumentError, /crazy/)
expect { engine.start }.to change(engine, :state).from(:stopped).to(:active)
{% end %}

Собираем всякие сложные штуки:

{% coderay %}
expect { engine.start }
  .to change(engine, :state).from(:stopped).to(:active)
  .and change(engine, :speed).to(1000)

expect(engine.internals)
  .to include('Carburator')
  .and have_attributes(gasoline: an_instance_of(Numeric))
{% end %}

## Группировка тестов: `describe` и `context`

{% coderay %}
describe Engine do
  its(:state) { is_expected.to eq :stopped }

  describe '#start' do
    context 'when engine has gasoline' do
      let(:gasoline) { 1_000 }
      it 'starts' do
        expect { engine.start }.not_to raise_error
      end
    end

    context 'when engine does not have gasoline' do
    end
  end
end
{% end %}

_(Тут можно выдохнуть и покурить)_

## RSpec и подходы к тестированию на Ruby

Дальше в программе:

* Как это всё работает;
* BDD: Behavior-Driven Development;
* Bottom-Up vs Top-Down testing;
* Advanced RSpec (если успеем);
* Другие средства тестирования (если успеем).

## Взгляд под капот

{% coderay %}
describe Engine do
  subject { described_class.new(gasoline) }

  context 'when engine has gasoline' do
    let(:gasoline) { 1_000 }

    it { is_expected.to be_full }

    it 'is stopped initially' do
      expect(engine.speed).to eq 0
    end

    it 'starts' do
      expect { engine.start }
        .not_to raise_error
        .and change(engine, :speed).to(1000)
    end
  end
end
{% end %}

## Подход: Behavior-Driven Design

* Test-Driven Development: пишем тесты ДО кода;
* Behavior-Driven Development: описываем поведение ДО написания кода;

Одно и то же?.. Да, но есть нюансы.

## Код как план

{% coderay %}
describe Plane do
  context 'when takeoff' do
    context 'when not enough speed' do
    end

    context 'when enough speed' do
      it 'takes off!'
    end
  end

  context 'when in flight' do
    it 'is stable'
    it 'spends gasoline'
    it 'controls position'
  end
end
{% end %}

## Подход к тестированию: Bottom-Up

Начинаем с деталей реализации.

{% coderay %}
describe 'Элерон' do
  it { должен быть беленький }
  it { должен подниматься }
  it { должен опускаться }
end

describe 'Крыло' do
  it { должно иметь элерон }
end
# ...
{% end %}

!SLIDE

{% coderay %}
# ...
describe 'Летательный аппарат' do
  it { должен надуваться гелием }
end
{% end %}

![](oops.jpg)

## Подход к тестированию: Top-Down

{% coderay %}
describe 'Летательный аппарат' do
  it { должен летать }
end

# реализуем метод «летать»:
  def fly
    speedup until liftable?
    change_angle
  end

  # или
  def fly
    fill_volume(:helium) until mass < air_mass(volume)
  end

# → получаем ошибки
# → воплощаем недостающий функционал
# → повторяем
{% end %}

## Top-Down: заглушки на ненаписанное

* Мок = mock (подделка) — объект, который ведёт себя как будто он правда есть;
* Стаб = stub (заглушка) — поддельное поведение (возможно, существующего) метода.

![](MockTurtle.jpg)

## Моки, стабы, дубли

{% coderay %}
let(:left_wing) { double }
let(:right_wing) { double }

let(:plane) { Plane.new(wings: [left_wing, right_wing] }

it 'uses elerons' do
  expect(left_wing).to receive(:lift_eleron).with(10)
  expect(right_wing).to receive(:lift_eleron).with(10)
  plane.up!
end
{% end %}

## И ещё: сложные моки

(Это не про самолёт уже)

* WebMock:

{% coderay %}
stub_request(:get, %r{en\.wikipedia\.org}).
  to_return(body: 'Купи слона!')

expect(get('https://en.wikipedia.org/wiki/Elephant')).to include('слон')
{% end %}

* VCR:

{% coderay %}
context 'when wikipedia queried', vcr: true do
  it 'extracts article text' do
    expect(get('https://en.wikipedia.org/wiki/Elephant')).to include('mastodons')
  end
end
{% end %}

## И ещё: фикстуры и фабрики

* Фикстуры: данные для тестирования, определённые раз и навсегда

{% coderay %}
let(:airport) { Airport.load_from_yaml('spec/fixtures/airport1.yml') }
{% end %}

* Фабрики: способ создания (полу)случайных данных для тестов

{% coderay %}
# в спеке
let(:airport) { build(:airport) }

# отдельно
Factory.define(:airport) do
  runways { rand(3..5).times.map { build(:runway) } }
  name { Faker.city }
  weather do
    temp { rand(-10..+20) }
    wind { rand(0..100) }
  end
end
{% end %}

## Top-Down: Discovery testing

Тесты для понимания, что нам вообще надо:
* описываем поведение «главной части»;
* делаем его работающим, заменяя заглушками всё важное;
* реализуем самые важные классы, подменяя ими заглушки;
* повторяем!

## Tips, tricks and mindfucks

![](mindblown.jpg)

## DRY: shared contexts

Плохо:

{% coderay %}
context 'when speed is below sonic' do
  let(:speed) { 500 }
  its(:speed) { is_expected.to eq 500 }
  its(:supersonic?) { is_expected.to eq false }
end

context 'when speed is above sonic' do
  let(:speed) { 2000 }
  its(:speed) { is_expected.to eq 2000 }
  its(:supersonic?) { is_expected.to eq true }
end
{% end %}

!SLIDE

Хорошо:

{% coderay %}
context 'when speed is below sonic' do
  it_should_behave_like 'speed examples', 500, false
end

context 'when speed is above sonic' do
  it_should_behave_like 'speed examples', 2000, true
end
{% end %}

## Readable: custom matchers

Плохо:

{% coderay %}
expect(engine).to be_active.and have_attributes(rpm: 1000).and not_be_exploding
{% end %}

Хорошо:

{% coderay %}
RSpec::Matchers.define :work_at_rpm do |expected|
  match do |engine|
    engine.active? && engine.rpm == expected && !engine.exploding?
  end
end

expect(engine).to work_at_rpm(1000)
{% end %}

## Знай свой инструмент

* `spec/spec_helper.rb`
* `rspec spec/my/specs.rb:123`
* `.rspec`
* `--seed`, `--bisect` и другие

## Другие инструменты тестирования: MiniTest

(«Замена» RSpec)

{% coderay %}
class TestPlane
  def setup
    @plane = Plane.new
  end

  def test_plane_can_fly
    assert_equal(@plane.speed, 1000)
  end
end
{% end %}

## Другие инструменты тестирования: Cucumber

Более высокоуровневое описание тестов.

{% code %}
Фича: Самолёт летает
  Сценарий: Взлёт
    Дано: Пилот заводит самолёт
    И: Разгоняется по взлётной полосе
    Когда: Он тянет штурвал на себя
    Тогда: Самолёт должен взлететь
{% end %}

...потом реализуем шаги...

{% coderay %}
step /(.+) заводит (.+)/ do |who, what|
  what.engine.activate!
end
{% end %}

## Собираем всё в кучку

* Тестирование — это хорошо!
* RSpec — это очень хорошо! Всё просто:
  * говорим, что `describe` что-то,
  * ...потом в каком-то `context`,
  * ...`it` ожидается что будет вести себя так-то
  * ...и это, в общем, всё
* Behavior-driven development наш путь,
  * ...и RSpec на нём ужасно полезен,
  * ...но не забывайте сначала описывать общее поведение, потом опускаться к деталям.
* Штук много, все они пригодятся.

## Домашнее задание

* Берём статью из Википедии про какого-нибудь зверя, напр. [слона](https://en.wikipedia.org/wiki/Elephant).
  Или [енота](https://en.wikipedia.org/wiki/Raccoon);
* Переписываем её в виде тестов на RSpec — начиная с базовых свойств зверя, затем ко всем
  его «компонентам»;
* Можно и нужно добавить custom matchers;
* Групповая работа над базовыми тестами зверей одобряется и поощряется (с пометкой «Вася разработал
  тесты для пищеварительной системы млекопитающих, использую её»), тупое списывание вызывает
  отвращение лектора;
* Реализация тестируемого зверя поощряется, но не обязательна; для «реализации» достаточно
  возвращать какие-то значения или выводить в консоль что-нибудь, не надо живых слонов
  приводить на лекцию (или 3D-модели оных воплощать).

# Спасибо!

Не переключайте канал.

Советы напоследок:

* [Читайте доки](http://rspec.info/documentation/) и [BetterSpecs](http://betterspecs.org/ru/);
* Полюбите разработку через тестирование;
* Тестируйте сверху вниз.

**Следующие лекции**:

* 10 ноября (четверг) — лекция по построению руби-гемов;
* 15 ноября (вторник) — Building web apps with Rack.

