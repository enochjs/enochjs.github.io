---
title: jest-api
date: 2020-08-25 14:53:29
tags:
---

### global
- beforeAll(fn, timeout)
- afterAll(fn, timeout)
- beforeEach(fn, timeout)
- afterEach(fn, timeout)
- describe
    > describe(name, fn)   
    > describe.each(table)(name, fn)
  - only 只运行当前语法块（一般调试有问题的测试模块时使用）
  - skip 跳过当前语法块 (遇到比较棘手的问题，又想把其他的测试跑完，可以先跳过)

- test(it) 
    > test(name, fn, timeout)  
    > test.each(table)(name, fn, timeout)
    > test.concurrent(name, fn, timeout) experimental
  - only 只运行当前测试
  - skip 跳过当前测试
  - todo 

  > **table** 
  > ```javascript
  1. array[]
  占位符
  %p - pretty-format.
  %s- String.
  %d- Number.
  %i - Integer.
  %f - Floating point value.
  %j - JSON.
  %o - Object.
  %# - Index of the test case.
  %% - single percent sign ('%'). This does not consume an argument.

  // example
  describe.each([
    [1, 1, 2],
    [1, 2, 3],
    [2, 1, 3],
  ])('.add(%i, %i)', (a, b, expected) => {
    test(`returns ${expected}`, () => {
      expect(a + b).toBe(expected);
    });

    test(`returned value not be greater than ${expected}`, () => {
      expect(a + b).not.toBeGreaterThan(expected);
    });

    test(`returned value not be less than ${expected}`, () => {
      expect(a + b).not.toBeLessThan(expected);
    });
  });

  2. template
  // example
  describe.each`
    a    | b    | expected
    ${1} | ${1} | ${2}
    ${1} | ${2} | ${3}
    ${2} | ${1} | ${3}
  `('$a + $b', ({a, b, expected}) => {
    test(`returns ${expected}`, () => {
      expect(a + b).toBe(expected);
    });

    test(`returned value not be greater than ${expected}`, () => {
      expect(a + b).not.toBeGreaterThan(expected);
    });

    test(`returned value not be less than ${expected}`, () => {
      expect(a + b).not.toBeLessThan(expected);
    });
  });
  ```
### expect
  expect(received).toEqual(expected)
  - received
    1. expect(value) except(value).not
  - matcher
    1. constom
      - api
     ```typescript
      expect.extend({
        yourMatcher(x, y, z) {
          return {
            pass: true, // false => test failed
            message: () => '',
          };
        },
      });
     ```
      - ts interface
    ```typescript
    declare global {
      namespace jest {
        interface Matchers<R> {
          toBeWithinRange(a: number, b: number): R;
          getExternalValueFromRemoteSource(received: number): Promise<R>
        }
      }
    }
    ```
      - example
    ```typescript
      expect.extend({
        toBeWithinRange(received, floor, ceiling) {
          const pass = received >= floor && received <= ceiling;
          if (pass) {
            return {
              message: () =>
                `expected ${received} not to be within range ${floor} - ${ceiling}`,
              pass: true,
            };
          } else {
            return {
              message: () =>
                `expected ${received} to be within range ${floor} - ${ceiling}`,
              pass: false,
            };
          }
        },
        async toBeDivisibleByExternalValue(received) {
          const externalValue = await getExternalValueFromRemoteSource();
          const pass = received % externalValue == 0;
          if (pass) {
            return {
              message: () =>
                `expected ${received} not to be divisible by ${externalValue}`,
              pass: true,
            };
          } else {
            return {
              message: () =>
                `expected ${received} to be divisible by ${externalValue}`,
              pass: false,
            };
          }
        },
      });

      test('numeric ranges', () => {
        expect(100).toBeWithinRange(90, 110);
        expect(101).not.toBeWithinRange(0, 100);
        expect({apples: 6, bananas: 3}).toEqual({
          apples: expect.toBeWithinRange(1, 10),
          bananas: expect.not.toBeWithinRange(11, 20),
        });
      });

      test('is divisible by external value', async () => {
        await expect(100).toBeDivisibleByExternalValue();
        await expect(101).not.toBeDivisibleByExternalValue();
      });
    ```
    2. system
      1. type
        - toBeDefined()
        - toBeTruthy()
        - toBeFalsy()
        - toBeInstanceOf(Class)
        - toBeNull()
        - toBeUndefined()
        - toBeNaN()
      2. compare
        - expect.anything()
          > 匹配非null和undefined的任意值
        - expect.any(constructor)
          > 匹配任意由constructor创建的值
        - toBe(value)
          > strict equailty ===
        - toBeCloseTo
          > 浮点数运算使用
        - toBeGreaterThan
        - toBeGreaterThanOrEqual
        - toBeLessThan
        - toBeLessThanOrEqual
      3. function
        - toHaveBeenCalled()
        - toHaveBeenCalledTimes(number)
        - toHaveBeenCalledWith(arg1, arg2, ...)
        - toHaveBeenLastCalledWith(arg1, arg2, ...)
        - toHaveBeenNthCalledWith(nthCall, arg1, arg2, ....)
          ```typescript
            test('drinkEach drinks each drink', () => {
              const drink = jest.fn();
              drinkEach(drink, ['lemon', 'octopus']);
              expect(drink).toHaveBeenNthCalledWith(1, 'lemon');
              expect(drink).toHaveBeenNthCalledWith(2, 'octopus');
            });
          ```
        - toHaveReturned()
        - toHaveReturnedTimes(number)
        - toHaveReturnedWith(value)
        - toHaveLastReturnedWith(value)
        - toHaveNthReturnedWith(nthCall, value)
          ```typescript
            test('drink returns expected nth calls', () => {
              const beverage1 = {name: 'La Croix (Lemon)'};
              const beverage2 = {name: 'La Croix (Orange)'};
              const drink = jest.fn(beverage => beverage.name);
              drink(beverage1);
              drink(beverage2);
              expect(drink).toHaveNthReturnedWith(1, 'La Croix (Lemon)');
              expect(drink).toHaveNthReturnedWith(2, 'La Croix (Orange)');
            });
          ```
      4. promise
        - resolves
          ```typescript
            test('resolves to lemon', () => {
              // make sure to add a return statement
              return expect(Promise.resolve('lemon')).resolves.toBe('lemon');
            });
            test('resolves to lemon', async () => {
              await expect(Promise.resolve('lemon')).resolves.toBe('lemon');
              await expect(Promise.resolve('lemon')).resolves.not.toBe('octopus');
            });
          ```
        - rejects
        - toThrow
          ```typescript
            test('rejects to octopus', () => {
              // make sure to add a return statement
              return expect(Promise.reject(new Error('octopus'))).rejects.toThrow(
                'octopus',
              );
            });
            test('rejects to octopus', async () => {
              await expect(Promise.reject(new Error('octopus'))).rejects.toThrow('octopus');
            });
          ```
      5. asserts
        - expect.assertions
          ```typescript
            // 断言执行的次数
            test('doAsync calls both callbacks', () => {
              expect.assertions(2);
              function callback1(data) {
                expect(data).toBeTruthy();
              }
              function callback2(data) {
                expect(data).toBeTruthy();
              }

              doAsync(callback1, callback2);
            });
          ```
        - expect.hasAssertions
      6. string
        - expect.stringContaining
        - expect.stringMatching
        - toHaveLength
        - toMatch
      7. object
        - expect.objectContaining
          > 包含
        - toHaveProperty(keyPath, value?)
        ```javascript
        const houseForSale = {
          bath: true,
          bedrooms: 4,
          kitchen: {
            amenities: ['oven', 'stove', 'washer'],
            area: 20,
            wallColor: 'white',
            'nice.oven': true,
          },
          'ceiling.height': 2,
        };
        test('this house has my desired features', () => {
          // Example Referencing
          expect(houseForSale).toHaveProperty('bath');
          expect(houseForSale).toHaveProperty('bedrooms', 4);
          // Deep referencing using dot notation
          expect(houseForSale).toHaveProperty('kitchen.area', 20);
          expect(houseForSale).toHaveProperty('kitchen.amenities', [
            'oven',
            'stove',
            'washer',
          ]);
          // Deep referencing using an array containing the keyPath
          expect(houseForSale).toHaveProperty(['kitchen', 'area'], 20);
          expect(houseForSale).toHaveProperty(
            ['kitchen', 'amenities'],
            ['oven', 'stove', 'washer'],
          );
          expect(houseForSale).toHaveProperty(['kitchen', 'amenities', 0], 'oven');
          expect(houseForSale).toHaveProperty(['kitchen', 'nice.oven']);
          expect(houseForSale).toHaveProperty(['ceiling.height'], 'tall');
        });
        ```
        - toEqual
        - toMatchObject
          > 包含
        - toStrictEqual
          > {a: undefined, b: 2} not toStrictEqual {b: 2}
      8. array
        - expect.arrayContaining(array)
          > 包含
        - toHaveLength
        - toContain
        - toContainEqual
          > 匹配所有值
        - toStrictEqual
          > [, 1] not toStrictEqual [undefined, 1]
      9. snapshot
        - expect.addSnapshotSerializer
        - toMatchSnapshot
        - toMatchInlineSnapshot
        - toThrowErrorMatchingSnapshot
        - toThrowErrorMatchingInlineSnapshot

### mocks
  1. mock modules
    - jest.disableAutomock()
      > 禁用automock；所有的 require() 都会返回真是的版本，而不是mock的版本
    - jest.enableAutomock()
    - jest.createMockFromModule(moduleName)
      > 从真实的文件mock，默认为require() 返回的值
      ```javascript
        // utils.js
        export default {
          authorize: () => {
            return 'token';
          },
          isAuthorized: secret => secret === 'wizard',
        };

        // __tests__/createMockFromModule.test.js
        const utils = jest.createMockFromModule('../utils').default;
        utils.isAuthorized = jest.fn(secret => secret === 'not wizard');

        test('implementation created by jest.createMockFromModule', () => {
          expect(utils.authorize.mock).toBeTruthy();
          expect(utils.authorize()).toBe('token');
          expect(utils.isAuthorized('not wizard')).toEqual(true);
        });
      ```
    - jest.mock(moduleName, factory, options)
      > 自动mock, 如果没有实现factory，将返回undefined
      ```javascript
        // banana.js
        module.exports = () => 'banana';
        // __tests__/test.js
        jest.mock('../banana');
        const banana = require('../banana'); // banana will be explicitly mocked.
        banana(); // return  undefined

        import moduleName, {foo} from '../moduleName';

        jest.mock('../moduleName', () => {
          return {
            __esModule: true,
            default: jest.fn(() => 42),
            foo: jest.fn(() => 43),
          };
        });
        moduleName(); // Will return 42
        foo(); // Will return 43
      ```
    - jest.unmock(moduleName)
      > 指定某个文件使用真是的moudle
    - jest.doMock(moduleName, factory, options)
      > jest.mock 是要放在top level的， doMock可以在语法块里面
      ```javascript
        beforeEach(() => {
          jest.resetModules();
        });

        test('moduleName 1', () => {
          jest.doMock('../moduleName', () => {
            return jest.fn(() => 1);
          });
          const moduleName = require('../moduleName');
          expect(moduleName()).toEqual(1);
        });

        test('moduleName 2', () => {
          jest.doMock('../moduleName', () => {
            return jest.fn(() => 2);
          });
          const moduleName = require('../moduleName');
          expect(moduleName()).toEqual(2);
        });
      ```
    - jest.dontMock(moduleName)
    - jest.setMock(moduleName, moduleExports)
    - jest.requireActual(moduleName)
    - jest.requireMock(moduleName)
    - jest.resetModules()
      > reset cache value
      ```javascript
        const sum1 = require('../sum');
        jest.resetModules();
        const sum2 = require('../sum');
        sum1 === sum2; // false
      ```
    - jest.isolateModules(fn)
      > 隔离moudle，类似于创建一个沙箱
      ```javascript
        let myModule;
        jest.isolateModules(() => {
          myModule = require('myModule');
        });

        const otherCopyOfMyModule = require('myModule');
      ```
  2. mock functions 
    - jest.fn(implementation)
    - jest.isMockFunction(fn)
    - jest.spyOn(object, methodName)
      ```javascript
        const video = {
          play() {
            return true;
          },
        };

        module.exports = video;

        // test
        const video = require('./video');

        test('plays video', () => {
          const spy = jest.spyOn(video, 'play');
          const isPlaying = video.play();

          expect(spy).toHaveBeenCalled();
          expect(isPlaying).toBe(true);

          spy.mockRestore();
        });
      ```
    - jest.spyOn(object, methodName, accessType?)
      > accessType set | get
    - jest.clearAllMocks()
      > 清除所有mocks 的 mock.calls mock.instances 属性
    - jest.resetAllMocks()
      > 清除所有mocks 的所有状态
    - jest.restoreAllMocks()
      > 返回到orgin的状态
    - mockFn
      - .getMockName()
      - .mock.calls
      ```javascript
        f('arg1', 'arg2')
        f('arg3', 'arg4')
        mock.calls => [['args1', 'args2'], ['args3', 'args4']]
      ```
      - .mock.results
        ```javascript
          [
            {
              type: 'return',
              value: 'result1',
            },
            {
              type: 'throw',
              value: {
                /* Error instance */
              },
            },
            {
              type: 'return',
              value: 'result2',
            },
          ];
        ```
      - .mock.instances
        ```javascript
        const mockFn = jest.fn();

        const a = new mockFn();
        const b = new mockFn();

        mockFn.mock.instances[0] === a; // true
        mockFn.mock.instances[1] === b; // true
        ```
      - .mockClear()
        > 清除 mockFn.mock.calls和mockFn.mock.instances
      - .mockReset()
        > 返回到初始化的状态
      - .mockRestore()
        > 返回到origin
      - .mockImplementation(fn)
        ```javascript
          const mockFn = jest.fn().mockImplementation(scalar => 42 + scalar);
          // or: jest.fn(scalar => 42 + scalar);

          const a = mockFn(0);
          const b = mockFn(1);

          a === 42; // true
          b === 43; // true

          mockFn.mock.calls[0][0] === 0; // true
          mockFn.mock.calls[1][0] === 1; // true
        ```
      - .mockImplementationOnce(fn)
        ```javascript
          const myMockFn = jest
            .fn(() => 'default')
            .mockImplementationOnce(() => 'first call')
            .mockImplementationOnce(() => 'second call');
          // 'first call', 'second call', 'default', 'default'
          console.log(myMockFn(), myMockFn(), myMockFn(), myMockFn());
        ```
      - .mockName(value)
        > 打印信息用
      - .mockReturnThis()
      - .mockReturnValue(value)
      - .mockReturnValueOnce(value)
        ```javascript
          const myMockFn = jest
            .fn()
            .mockReturnValue('default')
            .mockReturnValueOnce('first call')
            .mockReturnValueOnce('second call');
          // 'first call', 'second call', 'default', 'default'
          console.log(myMockFn(), myMockFn(), myMockFn(), myMockFn());
        ```
      - .mockResolvedValue(value)
        > jest.fn().mockImplementation(() => Promise.resolve(value));
        ```javascript
          test('async test', async () => {
            const asyncMock = jest.fn().mockResolvedValue(43);
            await asyncMock(); // 43
          });
        ```
      - .mockResolvedValueOnce(value)
        > jest.fn().mockImplementationOnce(() => Promise.resolve(value));
        ```javascript
          test('async test', async () => {
            const asyncMock = jest
              .fn()
              .mockResolvedValue('default')
              .mockResolvedValueOnce('first call')
              .mockResolvedValueOnce('second call');

            await asyncMock(); // first call
            await asyncMock(); // second call
            await asyncMock(); // default
            await asyncMock(); // default
          });
        ```
      - .mockRejectedValue(value)
        > jest.fn().mockImplementation(() => Promise.reject(value));
      - .mockRejectedValueOnce(value)
        > jest.fn().mockImplementationOnce(() => Promise.reject(value));
        ```javascript
          test('async test', async () => {
            const asyncMock = jest
              .fn()
              .mockResolvedValueOnce('first call')
              .mockRejectedValueOnce(new Error('Async error'));

            await asyncMock(); // first call
            await asyncMock(); // throws "Async error"
          });
        ```
  3. mock timers
    - jest.useFakeTimers(implementation?: 'modern' | 'legacy')
    - jest.useRealTimers()
    - jest.runAllTicks()
      > 执行所有的微任务 process.nextTick
    - jest.runAllTimers()
      > 执行所有的微任务和宏任务 process.nextTick setTimeout setInterval setImmediate
    - jest.runAllImmediates()
      > 执行所有的 setImmediate
    - jest.advanceTimersByTime(msToRun)
      > 当前时间提前 msToRun毫秒，比如一个任务需要1000ms后执行, jest.advanceTimersByTime(1000) 就会立即执行
      ```javascript
      // timerGame.js
      'use strict';

      function timerGame(callback) {
        console.log('Ready....go!');
        setTimeout(() => {
          console.log("Time's up -- stop!");
          callback && callback();
        }, 1000);
      }

      module.exports = timerGame;

      it('calls the callback after 1 second via advanceTimersByTime', () => {
        const timerGame = require('../timerGame');
        const callback = jest.fn();

        timerGame(callback);

        // At this point in time, the callback should not have been called yet
        expect(callback).not.toBeCalled();

        // Fast-forward until all timers have been executed
        jest.advanceTimersByTime(1000);

        // Now our callback should have been called!
        expect(callback).toBeCalled();
        expect(callback).toHaveBeenCalledTimes(1);
      });
      ```
    - jest.runOnlyPendingTimers()
      > 只执行已经 pending的task
      ```javascript
        // infiniteTimerGame.js
        'use strict';

        function infiniteTimerGame(callback) {
          console.log('Ready....go!');

          setTimeout(() => {
            console.log("Time's up! 10 seconds before the next game starts...");
            callback && callback();

            // Schedule the next game in 10 seconds
            setTimeout(() => {
              infiniteTimerGame(callback);
            }, 10000);
          }, 1000);
        }

        module.exports = infiniteTimerGame;

        // __tests__/infiniteTimerGame-test.js
        'use strict';

        jest.useFakeTimers();

        describe('infiniteTimerGame', () => {
          test('schedules a 10-second timer after 1 second', () => {
            const infiniteTimerGame = require('../infiniteTimerGame');
            const callback = jest.fn();

            infiniteTimerGame(callback);

            // At this point in time, there should have been a single call to
            // setTimeout to schedule the end of the game in 1 second.
            expect(setTimeout).toHaveBeenCalledTimes(1);
            expect(setTimeout).toHaveBeenLastCalledWith(expect.any(Function), 1000);

            // Fast forward and exhaust only currently pending timers
            // (but not any new timers that get created during that process)
            jest.runOnlyPendingTimers();

            // At this point, our 1-second timer should have fired it's callback
            expect(callback).toBeCalled();

            // And it should have created a new timer to start the game over in
            // 10 seconds
            expect(setTimeout).toHaveBeenCalledTimes(2);
            expect(setTimeout).toHaveBeenLastCalledWith(expect.any(Function), 10000);
          });
        });
      ```
    - jest.advanceTimersToNextTimer(steps)
    - jest.clearAllTimers()
    - jest.getTimerCount()
    - jest.setSystemTime(now?: number | Date)
    - jest.getRealSystemTime()
  4. misc
    - jest.setTimeout(timeout)
      ```javascript
        jest.setTimeout(1000);
      ```
    - jest.retryTimes()
      ```javascript
        jest.retryTimes(3);
        test('will fail', () => {
          expect(true).toBe(false);
        });
      ```

### jest文档
[jest](https://jestjs.io/docs/en/getting-started)
