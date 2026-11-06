# 🧪 Rails TDD & RSpec Standards

[![ClawHub Skill](https://img.shields.io/badge/ClawHub-Skill-blue)](https://clawhub.ai/djc00p/rails-tdd-standards) [![Agent Skill](https://img.shields.io/badge/Agent-Skill-blue)](#) [![Rails](https://img.shields.io/badge/Rails-v7+-red.svg)](https://rubyonrails.org/) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**A standardized protocol for writing clean, reliable, and maintainable RSpec tests within Ruby on Rails applications.**

The purpose of this standard is to eliminate "flaky" tests, reduce debugging time, and ensure that the test suite serves as live, executable documentation for the application's behavior. By following these patterns, we ensure that every test failure is meaningful and every test pass is trustworthy.

---

## 🎯 The Core Principle: Single Expectation

Every `it` block must verify exactly **one** behavior. If an `it` block contains multiple `expect` statements, it is no longer a specification; it is a script.

**Why?** When a multi-assertion test fails, you don't know which part of the logic broke without debugging. Single-assertion tests pinpoint the exact failure point immediately.

| ❌ **Anti-Pattern (Multiple Assertations)** | ✅ **Standard (Single Expectation)** |
| :---  | :--- |
| <pre>it "validates the user" do<br>  expect(user).to validate_presence_of(:email)<br>  expect(user).to validate_presence_of(:name)<br>  expect(user).to be_valid<br>end</pre> | <pre>it { is_expected.to validate_presence_of(:email) }</pre><pre>it { is_expected.to validate_presence_of(:name) }</pre><pre>it { is_expected.to be_valid }</pre> |

---

## 🏗️ FactoryBot implementation Standards

Improperly configured factories are the #1 cause of `ActiveRecord::RecordInvalid` errors in Rails test suites.

### 1. Mandatory Role-Based Traits

Never instantiate a user without defining their role. Use **traits** to ensure the object has the correct permissions and state from the moment of creation.

* **Bad:** `create(:user)` (Results in an ambiguous, potentially invalid user)
* **Good:** `create(:user, :admin)` or `create(:user, :driver)`

### 2. Explicit Association Keys

Always verify that the key passed to the factory matches the association name defined in the factory file. Silent failures occur when developers pass `user: user` to a factory expecting `owner: user`.

### 3. The "No-Assumption" Rule for Associations

Do not rely on factory defaults for required associations. If a model requires a `profile`, explicitly pass the profile during creation. This makes the dependency visible to anyone reading the test.

```ruby
# ✅ Explicit and Unambiguous
let(:record) { create(:model, required_association: other_record) }

# ❌ Implicit and Fragile (Breaks if factory defaults change)
let(:record) { create(:model) }
```

---

## 📐 Test Suite Architecture

Tests should follow a hierarchical structure that mirrors the application's logic: `Describe` (The Class) $\rightarrow$ `Describe` (The Method) $\rightarrow$ `Context` (The State).

### The Standard Template

```ruby
Rspec.describe MyService do
  # 1. Use described_class to avoid hardcoding class names
  subject(:service) { described_class.new(params) }

  # 2. Define shared setup via 'let'
  let(:user) { create(:user, :admin) }
  let(:params) { { user: user, amount: 100 } }

  # 3. Group by method name
  describe "#execute" do
    # 4. Use 'context' to define specific environmental states
    context "when the account has sufficient funds" do
      it "compleget the transaction" do
        expect(service.execute).to be_success
      end
    end

    context "when the account is overdrawn" do
      let(:params) { { user: user, amount: 999_999 } }
      
      it "raises a transaction error" do
        expect { service.execute }.to raise_error(InsufficientFundsError)
      end
    end
  end
end
```

---

## 🛠️ Mocking, Stubbing, and WebMock

To keep tests fast and deterministic, external dependencies must be isolated.

* **Method Stubbing:** Use `allow(object).to receive(:method).and_return(value)` for internal logic.
* **Behavior Verification:** Use `expect(object).to receive(:method).once` to ensure side effects occur.
* **Network Isolation:** Use **WebMock** to intercept all outgoing HTTP requests.
  * *Rule:* Always `WebMock.disable_net_connect!(allow_localhost: true)` to prevent tests from accidentally hitting real external APIs.

---

## 🚀 Execution & Debugging

### Essential CLI Commands

| Command | Purpose |
| :--- | :--- |
| `bundle exec rspec` | Run the entire test suite. |
| `bundle exec rspec spec/models/user_spec.rb` | Run a single file. |
| `bundle exec rspec spec/models/user_spec.rb:42` | Run a single test line (Critical for debugging). |
  | `bundle exec rspec --only-failures` | Run only the tests that failed in the last session. |
| `bundle exec rspec --format documentation` | Print tests in a readable, human-friendly tree. |

### Troubleshooting `RecordInvalid`

If you encounter `ActiveRecord::RecordInvalid: Validation failed`, check the following:

1. **Missing Traits:** Does the factory require a `:role` that was omitted?
2. **Association Mismatch:** Are you passing `user_id` instead of the `user` object?
3. **Database Constraints:** Is a `NOT NULL` constraint being violated by a nil factory attribute?

---
**Standard maintained by the Engineering Team.**
