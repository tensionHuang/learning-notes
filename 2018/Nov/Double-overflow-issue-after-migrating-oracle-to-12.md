### Double overflow issue after migrating oracle to 12
#### 描述：
在升級 Oracle db 從 11 到 12 之後，因為 11 之前用的 oracle jdbc 是 10 的版本，現在換到 oracle jdbc 12.1.0.2 的版本，出現下列錯誤:

```
Caused by:
java.lang.IllegalArgumentException: Overflow
        at oracle.jdbc.driver.OraclePreparedStatement.setDoubleInternal(OraclePreparedStatement.java:6607)
        at oracle.jdbc.driver.OraclePreparedStatement.setDouble(OraclePreparedStatement.java:6574)
        at oracle.jdbc.driver.OraclePreparedStatementWrapper.setDouble(OraclePreparedStatementWrapper.java:193)
        at com.mchange.v2.c3p0.impl.NewProxyPreparedStatement.setDouble(NewProxyPreparedStatement.java:755)
......
```

這發生於在試圖儲存某個 double 欄位的值是 `Infinity` or `NaN` 會出現的錯誤。

在 trace oracle jdbc 12.1.0.2 的 OraclePreparedStatement.class 之後，可以看到這個版本開始會對
像是 `Infinity`, `NaN`, ... 做檢查。

```
    void setDoubleInternal(int var1, double var2) throws SQLException {
        int var4 = var1 - 1;
        if (var4 >= 0 && var1 <= this.numberOfBindPositions) {
            if (!this.connection.setFloatAndDoubleUseBinary) {
                if (Double.isNaN(var2)) {
                    throw new IllegalArgumentException("NaN");
                }

                double var7 = Math.abs(var2);
                if (var7 != 0.0D && var7 < 1.0E-130D) {
                    throw new IllegalArgumentException("Underflow");
                }

                if (var7 >= 1.0E126D) {
                    throw new IllegalArgumentException("Overflow");
                }
            }
            
            // other code ... just skip them here
        }
    }

``` 

其實從 oracle jdbc 11 的某個版本開始就換成這個實作了...

#### 解決方法：
* 如果要繼續可以允許儲存 `Infinity`, `NaN` 到 DB，他有一個 flag `this.connection.setFloatAndDoubleUseBinary` 可以控制要不要檢查。
* 在執行 java 的時候，加入這個設定 `-Doracle.jdbc.SetFloatAndDoubleUseBinary=true` 就可以跳過檢查，也就可以和原本的版本相容。

