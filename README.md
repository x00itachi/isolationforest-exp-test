```python
import pandas as pd
from sklearn.ensemble import IsolationForest
from sklearn import preprocessing
import string
from urllib.parse import urlparse, parse_qs

```


```python
columns_str = 'frame.number,frame.len,frame.time,frame.time_epoch,frame.protocols,eth.src,eth.dst,eth.type,ip.src,ip.dst,ip.len,ip.ttl,ip.flags,ip.frag_offset,ip.proto,ip.version,ip.dsfield,ip.checksum,tcp.srcport,tcp.dstport,tcp.len,tcp.seq,tcp.ack,tcp.flags,tcp.flags.syn,tcp.flags.ack,tcp.flags.fin,tcp.flags.reset,tcp.window_size,tcp.checksum,tcp.stream,udp.srcport,udp.dstport,udp.length,udp.checksum,icmp.type,icmp.code,icmp.checksum,http.request.method,http.request.uri,http.request.version,http.request.full_uri,http.response.code,http.user_agent,http.content_length_header,http.content_type,http.cookie,http.host,http.referer,http.location,http.authorization,http.connection,dns.qry.name,dns.qry.type,dns.qry.class,dns.flags.response,dns.flags.recdesired,dns.flags.rcode,dns.resp.ttl,dns.resp.len,smtp.req.command,smtp.data.fragment,pop.request.command,pop.response,imap.request.command,imap.response,ftp.request.command,ftp.request.arg,ftp.response.code,ftp.response.arg,ipv6.src,ipv6.dst,ipv6.plen,alert'
data_columns = columns_str.split(',')
data_columns
```




    ['frame.number',
     'frame.len',
     'frame.time',
     'frame.time_epoch',
     'frame.protocols',
     'eth.src',
     'eth.dst',
     'eth.type',
     'ip.src',
     'ip.dst',
     'ip.len',
     'ip.ttl',
     'ip.flags',
     'ip.frag_offset',
     'ip.proto',
     'ip.version',
     'ip.dsfield',
     'ip.checksum',
     'tcp.srcport',
     'tcp.dstport',
     'tcp.len',
     'tcp.seq',
     'tcp.ack',
     'tcp.flags',
     'tcp.flags.syn',
     'tcp.flags.ack',
     'tcp.flags.fin',
     'tcp.flags.reset',
     'tcp.window_size',
     'tcp.checksum',
     'tcp.stream',
     'udp.srcport',
     'udp.dstport',
     'udp.length',
     'udp.checksum',
     'icmp.type',
     'icmp.code',
     'icmp.checksum',
     'http.request.method',
     'http.request.uri',
     'http.request.version',
     'http.request.full_uri',
     'http.response.code',
     'http.user_agent',
     'http.content_length_header',
     'http.content_type',
     'http.cookie',
     'http.host',
     'http.referer',
     'http.location',
     'http.authorization',
     'http.connection',
     'dns.qry.name',
     'dns.qry.type',
     'dns.qry.class',
     'dns.flags.response',
     'dns.flags.recdesired',
     'dns.flags.rcode',
     'dns.resp.ttl',
     'dns.resp.len',
     'smtp.req.command',
     'smtp.data.fragment',
     'pop.request.command',
     'pop.response',
     'imap.request.command',
     'imap.response',
     'ftp.request.command',
     'ftp.request.arg',
     'ftp.response.code',
     'ftp.response.arg',
     'ipv6.src',
     'ipv6.dst',
     'ipv6.plen',
     'alert']




```python
df = pd.read_csv('attack-simulation-http-url.csv', usecols = ['http.request.uri'], names=data_columns, header=None, low_memory=False)
df.head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>http.request.uri</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>/dvwa/config/</td>
    </tr>
    <tr>
      <th>1</th>
      <td>/dvwa/docs/</td>
    </tr>
    <tr>
      <th>2</th>
      <td>/dvwa/external/</td>
    </tr>
    <tr>
      <th>3</th>
      <td>/mutillidae/ajax/</td>
    </tr>
    <tr>
      <th>4</th>
      <td>/mutillidae/classes/</td>
    </tr>
    <tr>
      <th>5</th>
      <td>/mutillidae/data/</td>
    </tr>
    <tr>
      <th>6</th>
      <td>/mutillidae/documentation/</td>
    </tr>
    <tr>
      <th>7</th>
      <td>/mutillidae/images/</td>
    </tr>
    <tr>
      <th>8</th>
      <td>/mutillidae/includes/</td>
    </tr>
    <tr>
      <th>9</th>
      <td>/mutillidae/javascript/</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Preprocessing

def uri_len(uri):
    uri_str = str(uri)
    return len(uri_str)

df['uri_len'] = df['http.request.uri'].apply(uri_len)
df.head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>http.request.uri</th>
      <th>uri_len</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>/dvwa/config/</td>
      <td>13</td>
    </tr>
    <tr>
      <th>1</th>
      <td>/dvwa/docs/</td>
      <td>11</td>
    </tr>
    <tr>
      <th>2</th>
      <td>/dvwa/external/</td>
      <td>15</td>
    </tr>
    <tr>
      <th>3</th>
      <td>/mutillidae/ajax/</td>
      <td>17</td>
    </tr>
    <tr>
      <th>4</th>
      <td>/mutillidae/classes/</td>
      <td>20</td>
    </tr>
    <tr>
      <th>5</th>
      <td>/mutillidae/data/</td>
      <td>17</td>
    </tr>
    <tr>
      <th>6</th>
      <td>/mutillidae/documentation/</td>
      <td>26</td>
    </tr>
    <tr>
      <th>7</th>
      <td>/mutillidae/images/</td>
      <td>19</td>
    </tr>
    <tr>
      <th>8</th>
      <td>/mutillidae/includes/</td>
      <td>21</td>
    </tr>
    <tr>
      <th>9</th>
      <td>/mutillidae/javascript/</td>
      <td>23</td>
    </tr>
  </tbody>
</table>
</div>




```python
def count_uri_segments(uri):
    parsed_uri = urlparse(str(uri))
    path = parsed_uri.path.strip('/')  # Remove leading and trailing slashes
    segments = path.split('/')
    return len(segments)

df['uri_segs'] = df['http.request.uri'].apply(count_uri_segments)
df.head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>http.request.uri</th>
      <th>uri_len</th>
      <th>uri_segs</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>/dvwa/config/</td>
      <td>13</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>/dvwa/docs/</td>
      <td>11</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>/dvwa/external/</td>
      <td>15</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>/mutillidae/ajax/</td>
      <td>17</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>/mutillidae/classes/</td>
      <td>20</td>
      <td>2</td>
    </tr>
    <tr>
      <th>5</th>
      <td>/mutillidae/data/</td>
      <td>17</td>
      <td>2</td>
    </tr>
    <tr>
      <th>6</th>
      <td>/mutillidae/documentation/</td>
      <td>26</td>
      <td>2</td>
    </tr>
    <tr>
      <th>7</th>
      <td>/mutillidae/images/</td>
      <td>19</td>
      <td>2</td>
    </tr>
    <tr>
      <th>8</th>
      <td>/mutillidae/includes/</td>
      <td>21</td>
      <td>2</td>
    </tr>
    <tr>
      <th>9</th>
      <td>/mutillidae/javascript/</td>
      <td>23</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>




```python
def count_special_chars(uri):
    special_chars = []
    for char in str(uri):
        if char not in string.ascii_letters + string.digits + '/':
                    special_chars.append(char)
    return len(special_chars)

df['uri_s_chars'] = df['http.request.uri'].apply(count_special_chars)
df.head(30)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>http.request.uri</th>
      <th>uri_len</th>
      <th>uri_segs</th>
      <th>uri_s_chars</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>/dvwa/config/</td>
      <td>13</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>/dvwa/docs/</td>
      <td>11</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>/dvwa/external/</td>
      <td>15</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>/mutillidae/ajax/</td>
      <td>17</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>/mutillidae/classes/</td>
      <td>20</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>/mutillidae/data/</td>
      <td>17</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>/mutillidae/documentation/</td>
      <td>26</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>/mutillidae/images/</td>
      <td>19</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>/mutillidae/includes/</td>
      <td>21</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>/mutillidae/javascript/</td>
      <td>23</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>/mutillidae/passwords/</td>
      <td>22</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>/mutillidae/phpmyadmin/</td>
      <td>23</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>12</th>
      <td>/mutillidae/styles/</td>
      <td>19</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>13</th>
      <td>/mutillidae/test/</td>
      <td>17</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>14</th>
      <td>/mutillidae/webservices/</td>
      <td>24</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>15</th>
      <td>/mutillidae/phpmyadmin/examples/</td>
      <td>32</td>
      <td>3</td>
      <td>0</td>
    </tr>
    <tr>
      <th>16</th>
      <td>/mutillidae/phpmyadmin/js/</td>
      <td>26</td>
      <td>3</td>
      <td>0</td>
    </tr>
    <tr>
      <th>17</th>
      <td>/mutillidae/phpmyadmin/locale/</td>
      <td>30</td>
      <td>3</td>
      <td>0</td>
    </tr>
    <tr>
      <th>18</th>
      <td>/mutillidae/phpmyadmin/setup/</td>
      <td>29</td>
      <td>3</td>
      <td>0</td>
    </tr>
    <tr>
      <th>19</th>
      <td>/mutillidae/phpmyadmin/themes/</td>
      <td>30</td>
      <td>3</td>
      <td>0</td>
    </tr>
    <tr>
      <th>20</th>
      <td>/assets/</td>
      <td>8</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>21</th>
      <td>/cgi-bin/</td>
      <td>9</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>22</th>
      <td>/evil/</td>
      <td>6</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>23</th>
      <td>/gallery2/</td>
      <td>10</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>24</th>
      <td>NaN</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>25</th>
      <td>/gallery2/main.php</td>
      <td>18</td>
      <td>2</td>
      <td>1</td>
    </tr>
    <tr>
      <th>26</th>
      <td>/icon/</td>
      <td>6</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>27</th>
      <td>/images/</td>
      <td>8</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>28</th>
      <td>/javascript/</td>
      <td>12</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>29</th>
      <td>/joomla/</td>
      <td>8</td>
      <td>1</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
def check_reserved_characters(uri):
    parsed_uri = urlparse(str(uri))
    path = parsed_uri.path.strip('/')  # Remove leading and trailing slashes
    reserved_chars = "!*'();:@&=+$,/?#[]"
    reserved_char_count = sum(c in reserved_chars for c in path)
    return reserved_char_count

df['uri_res_chars'] = df['http.request.uri'].apply(check_reserved_characters)
df.head(30)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>http.request.uri</th>
      <th>uri_len</th>
      <th>uri_segs</th>
      <th>uri_s_chars</th>
      <th>uri_res_chars</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>/dvwa/config/</td>
      <td>13</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>/dvwa/docs/</td>
      <td>11</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>/dvwa/external/</td>
      <td>15</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>/mutillidae/ajax/</td>
      <td>17</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>/mutillidae/classes/</td>
      <td>20</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>/mutillidae/data/</td>
      <td>17</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>6</th>
      <td>/mutillidae/documentation/</td>
      <td>26</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7</th>
      <td>/mutillidae/images/</td>
      <td>19</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>8</th>
      <td>/mutillidae/includes/</td>
      <td>21</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>9</th>
      <td>/mutillidae/javascript/</td>
      <td>23</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>10</th>
      <td>/mutillidae/passwords/</td>
      <td>22</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>11</th>
      <td>/mutillidae/phpmyadmin/</td>
      <td>23</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>12</th>
      <td>/mutillidae/styles/</td>
      <td>19</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>13</th>
      <td>/mutillidae/test/</td>
      <td>17</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>14</th>
      <td>/mutillidae/webservices/</td>
      <td>24</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15</th>
      <td>/mutillidae/phpmyadmin/examples/</td>
      <td>32</td>
      <td>3</td>
      <td>0</td>
      <td>2</td>
    </tr>
    <tr>
      <th>16</th>
      <td>/mutillidae/phpmyadmin/js/</td>
      <td>26</td>
      <td>3</td>
      <td>0</td>
      <td>2</td>
    </tr>
    <tr>
      <th>17</th>
      <td>/mutillidae/phpmyadmin/locale/</td>
      <td>30</td>
      <td>3</td>
      <td>0</td>
      <td>2</td>
    </tr>
    <tr>
      <th>18</th>
      <td>/mutillidae/phpmyadmin/setup/</td>
      <td>29</td>
      <td>3</td>
      <td>0</td>
      <td>2</td>
    </tr>
    <tr>
      <th>19</th>
      <td>/mutillidae/phpmyadmin/themes/</td>
      <td>30</td>
      <td>3</td>
      <td>0</td>
      <td>2</td>
    </tr>
    <tr>
      <th>20</th>
      <td>/assets/</td>
      <td>8</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>21</th>
      <td>/cgi-bin/</td>
      <td>9</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>22</th>
      <td>/evil/</td>
      <td>6</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>23</th>
      <td>/gallery2/</td>
      <td>10</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>24</th>
      <td>NaN</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>25</th>
      <td>/gallery2/main.php</td>
      <td>18</td>
      <td>2</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>26</th>
      <td>/icon/</td>
      <td>6</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>27</th>
      <td>/images/</td>
      <td>8</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>28</th>
      <td>/javascript/</td>
      <td>12</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>29</th>
      <td>/joomla/</td>
      <td>8</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
def count_query_parameters(uri):
    parsed_uri = urlparse(str(uri))
    query_params = parsed_uri.query
    parsed_query_params = parse_qs(query_params)
    num_query_params = len(parsed_query_params)
    return num_query_params

df['uri_q_params'] = df['http.request.uri'].apply(count_query_parameters)
df.tail(30)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>http.request.uri</th>
      <th>uri_len</th>
      <th>uri_segs</th>
      <th>uri_s_chars</th>
      <th>uri_res_chars</th>
      <th>uri_q_params</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1964477</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>139</td>
      <td>2</td>
      <td>18</td>
      <td>1</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1964478</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>128</td>
      <td>2</td>
      <td>17</td>
      <td>1</td>
      <td>6</td>
    </tr>
    <tr>
      <th>1964479</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>123</td>
      <td>2</td>
      <td>17</td>
      <td>1</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1964480</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>120</td>
      <td>2</td>
      <td>15</td>
      <td>1</td>
      <td>6</td>
    </tr>
    <tr>
      <th>1964481</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>128</td>
      <td>2</td>
      <td>18</td>
      <td>1</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1964482</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>149</td>
      <td>2</td>
      <td>19</td>
      <td>1</td>
      <td>6</td>
    </tr>
    <tr>
      <th>1964483</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>120</td>
      <td>2</td>
      <td>16</td>
      <td>1</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1964484</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>154</td>
      <td>2</td>
      <td>20</td>
      <td>1</td>
      <td>6</td>
    </tr>
    <tr>
      <th>1964485</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>149</td>
      <td>2</td>
      <td>20</td>
      <td>1</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1964486</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>136</td>
      <td>2</td>
      <td>19</td>
      <td>1</td>
      <td>6</td>
    </tr>
    <tr>
      <th>1964487</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>154</td>
      <td>2</td>
      <td>21</td>
      <td>1</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1964488</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>141</td>
      <td>2</td>
      <td>20</td>
      <td>1</td>
      <td>6</td>
    </tr>
    <tr>
      <th>1964489</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>110</td>
      <td>2</td>
      <td>14</td>
      <td>1</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1964490</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>133</td>
      <td>2</td>
      <td>18</td>
      <td>1</td>
      <td>6</td>
    </tr>
    <tr>
      <th>1964491</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>115</td>
      <td>2</td>
      <td>15</td>
      <td>1</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1964492</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>162</td>
      <td>2</td>
      <td>22</td>
      <td>1</td>
      <td>6</td>
    </tr>
    <tr>
      <th>1964493</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>107</td>
      <td>2</td>
      <td>13</td>
      <td>1</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1964494</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>167</td>
      <td>2</td>
      <td>23</td>
      <td>1</td>
      <td>6</td>
    </tr>
    <tr>
      <th>1964495</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>136</td>
      <td>2</td>
      <td>17</td>
      <td>1</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1964496</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>123</td>
      <td>2</td>
      <td>16</td>
      <td>1</td>
      <td>6</td>
    </tr>
    <tr>
      <th>1964497</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>141</td>
      <td>2</td>
      <td>18</td>
      <td>1</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1964498</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>128</td>
      <td>2</td>
      <td>17</td>
      <td>1</td>
      <td>6</td>
    </tr>
    <tr>
      <th>1964499</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>127</td>
      <td>2</td>
      <td>17</td>
      <td>1</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1964500</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>120</td>
      <td>2</td>
      <td>15</td>
      <td>1</td>
      <td>6</td>
    </tr>
    <tr>
      <th>1964501</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>132</td>
      <td>2</td>
      <td>18</td>
      <td>1</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1964502</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>149</td>
      <td>2</td>
      <td>19</td>
      <td>1</td>
      <td>6</td>
    </tr>
    <tr>
      <th>1964503</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>124</td>
      <td>2</td>
      <td>16</td>
      <td>1</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1964504</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>154</td>
      <td>2</td>
      <td>20</td>
      <td>1</td>
      <td>6</td>
    </tr>
    <tr>
      <th>1964505</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>153</td>
      <td>2</td>
      <td>20</td>
      <td>1</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1964506</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>158</td>
      <td>2</td>
      <td>21</td>
      <td>1</td>
      <td>5</td>
    </tr>
  </tbody>
</table>
</div>




```python
def max_query_param_length(uri):
    parsed_uri = urlparse(str(uri))
    query_params = parsed_uri.query
    parsed_query_params = parse_qs(query_params)
    
    max_param_length = 0
    
    for param, values in parsed_query_params.items():
        for value in values:
            max_param_length = max(max_param_length, len(value))
    
    return max_param_length

df['uri_maxq_len'] = df['http.request.uri'].apply(max_query_param_length)
df.tail(30)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>http.request.uri</th>
      <th>uri_len</th>
      <th>uri_segs</th>
      <th>uri_s_chars</th>
      <th>uri_res_chars</th>
      <th>uri_q_params</th>
      <th>uri_maxq_len</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1964477</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>139</td>
      <td>2</td>
      <td>18</td>
      <td>1</td>
      <td>5</td>
      <td>38</td>
    </tr>
    <tr>
      <th>1964478</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>128</td>
      <td>2</td>
      <td>17</td>
      <td>1</td>
      <td>6</td>
      <td>18</td>
    </tr>
    <tr>
      <th>1964479</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>123</td>
      <td>2</td>
      <td>17</td>
      <td>1</td>
      <td>5</td>
      <td>22</td>
    </tr>
    <tr>
      <th>1964480</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>120</td>
      <td>2</td>
      <td>15</td>
      <td>1</td>
      <td>6</td>
      <td>14</td>
    </tr>
    <tr>
      <th>1964481</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>128</td>
      <td>2</td>
      <td>18</td>
      <td>1</td>
      <td>5</td>
      <td>25</td>
    </tr>
    <tr>
      <th>1964482</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>149</td>
      <td>2</td>
      <td>19</td>
      <td>1</td>
      <td>6</td>
      <td>35</td>
    </tr>
    <tr>
      <th>1964483</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>120</td>
      <td>2</td>
      <td>16</td>
      <td>1</td>
      <td>5</td>
      <td>21</td>
    </tr>
    <tr>
      <th>1964484</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>154</td>
      <td>2</td>
      <td>20</td>
      <td>1</td>
      <td>6</td>
      <td>38</td>
    </tr>
    <tr>
      <th>1964485</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>149</td>
      <td>2</td>
      <td>20</td>
      <td>1</td>
      <td>5</td>
      <td>42</td>
    </tr>
    <tr>
      <th>1964486</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>136</td>
      <td>2</td>
      <td>19</td>
      <td>1</td>
      <td>6</td>
      <td>22</td>
    </tr>
    <tr>
      <th>1964487</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>154</td>
      <td>2</td>
      <td>21</td>
      <td>1</td>
      <td>5</td>
      <td>45</td>
    </tr>
    <tr>
      <th>1964488</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>141</td>
      <td>2</td>
      <td>20</td>
      <td>1</td>
      <td>6</td>
      <td>25</td>
    </tr>
    <tr>
      <th>1964489</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>110</td>
      <td>2</td>
      <td>14</td>
      <td>1</td>
      <td>5</td>
      <td>15</td>
    </tr>
    <tr>
      <th>1964490</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>133</td>
      <td>2</td>
      <td>18</td>
      <td>1</td>
      <td>6</td>
      <td>21</td>
    </tr>
    <tr>
      <th>1964491</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>115</td>
      <td>2</td>
      <td>15</td>
      <td>1</td>
      <td>5</td>
      <td>18</td>
    </tr>
    <tr>
      <th>1964492</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>162</td>
      <td>2</td>
      <td>22</td>
      <td>1</td>
      <td>6</td>
      <td>42</td>
    </tr>
    <tr>
      <th>1964493</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>107</td>
      <td>2</td>
      <td>13</td>
      <td>1</td>
      <td>5</td>
      <td>14</td>
    </tr>
    <tr>
      <th>1964494</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>167</td>
      <td>2</td>
      <td>23</td>
      <td>1</td>
      <td>6</td>
      <td>45</td>
    </tr>
    <tr>
      <th>1964495</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>136</td>
      <td>2</td>
      <td>17</td>
      <td>1</td>
      <td>5</td>
      <td>35</td>
    </tr>
    <tr>
      <th>1964496</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>123</td>
      <td>2</td>
      <td>16</td>
      <td>1</td>
      <td>6</td>
      <td>15</td>
    </tr>
    <tr>
      <th>1964497</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>141</td>
      <td>2</td>
      <td>18</td>
      <td>1</td>
      <td>5</td>
      <td>38</td>
    </tr>
    <tr>
      <th>1964498</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>128</td>
      <td>2</td>
      <td>17</td>
      <td>1</td>
      <td>6</td>
      <td>18</td>
    </tr>
    <tr>
      <th>1964499</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>127</td>
      <td>2</td>
      <td>17</td>
      <td>1</td>
      <td>5</td>
      <td>22</td>
    </tr>
    <tr>
      <th>1964500</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>120</td>
      <td>2</td>
      <td>15</td>
      <td>1</td>
      <td>6</td>
      <td>14</td>
    </tr>
    <tr>
      <th>1964501</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>132</td>
      <td>2</td>
      <td>18</td>
      <td>1</td>
      <td>5</td>
      <td>25</td>
    </tr>
    <tr>
      <th>1964502</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>149</td>
      <td>2</td>
      <td>19</td>
      <td>1</td>
      <td>6</td>
      <td>35</td>
    </tr>
    <tr>
      <th>1964503</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>124</td>
      <td>2</td>
      <td>16</td>
      <td>1</td>
      <td>5</td>
      <td>21</td>
    </tr>
    <tr>
      <th>1964504</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>154</td>
      <td>2</td>
      <td>20</td>
      <td>1</td>
      <td>6</td>
      <td>38</td>
    </tr>
    <tr>
      <th>1964505</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>153</td>
      <td>2</td>
      <td>20</td>
      <td>1</td>
      <td>5</td>
      <td>42</td>
    </tr>
    <tr>
      <th>1964506</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>158</td>
      <td>2</td>
      <td>21</td>
      <td>1</td>
      <td>5</td>
      <td>45</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Drop rows with NaN values
df.dropna(inplace=True)
df

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>http.request.uri</th>
      <th>uri_len</th>
      <th>uri_segs</th>
      <th>uri_s_chars</th>
      <th>uri_res_chars</th>
      <th>uri_q_params</th>
      <th>uri_maxq_len</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>/dvwa/config/</td>
      <td>13</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>/dvwa/docs/</td>
      <td>11</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>/dvwa/external/</td>
      <td>15</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>/mutillidae/ajax/</td>
      <td>17</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>/mutillidae/classes/</td>
      <td>20</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1964502</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>149</td>
      <td>2</td>
      <td>19</td>
      <td>1</td>
      <td>6</td>
      <td>35</td>
    </tr>
    <tr>
      <th>1964503</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>124</td>
      <td>2</td>
      <td>16</td>
      <td>1</td>
      <td>5</td>
      <td>21</td>
    </tr>
    <tr>
      <th>1964504</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>154</td>
      <td>2</td>
      <td>20</td>
      <td>1</td>
      <td>6</td>
      <td>38</td>
    </tr>
    <tr>
      <th>1964505</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>153</td>
      <td>2</td>
      <td>20</td>
      <td>1</td>
      <td>5</td>
      <td>42</td>
    </tr>
    <tr>
      <th>1964506</th>
      <td>/awstats/awstats.pl?config=owaspbwa&amp;framename=...</td>
      <td>158</td>
      <td>2</td>
      <td>21</td>
      <td>1</td>
      <td>5</td>
      <td>45</td>
    </tr>
  </tbody>
</table>
<p>1914976 rows × 7 columns</p>
</div>




```python
# Create an Isolation Forest model
model=IsolationForest(contamination=0.1)
model

```




<style>#sk-container-id-1 {color: black;}#sk-container-id-1 pre{padding: 0;}#sk-container-id-1 div.sk-toggleable {background-color: white;}#sk-container-id-1 label.sk-toggleable__label {cursor: pointer;display: block;width: 100%;margin-bottom: 0;padding: 0.3em;box-sizing: border-box;text-align: center;}#sk-container-id-1 label.sk-toggleable__label-arrow:before {content: "▸";float: left;margin-right: 0.25em;color: #696969;}#sk-container-id-1 label.sk-toggleable__label-arrow:hover:before {color: black;}#sk-container-id-1 div.sk-estimator:hover label.sk-toggleable__label-arrow:before {color: black;}#sk-container-id-1 div.sk-toggleable__content {max-height: 0;max-width: 0;overflow: hidden;text-align: left;background-color: #f0f8ff;}#sk-container-id-1 div.sk-toggleable__content pre {margin: 0.2em;color: black;border-radius: 0.25em;background-color: #f0f8ff;}#sk-container-id-1 input.sk-toggleable__control:checked~div.sk-toggleable__content {max-height: 200px;max-width: 100%;overflow: auto;}#sk-container-id-1 input.sk-toggleable__control:checked~label.sk-toggleable__label-arrow:before {content: "▾";}#sk-container-id-1 div.sk-estimator input.sk-toggleable__control:checked~label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-1 div.sk-label input.sk-toggleable__control:checked~label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-1 input.sk-hidden--visually {border: 0;clip: rect(1px 1px 1px 1px);clip: rect(1px, 1px, 1px, 1px);height: 1px;margin: -1px;overflow: hidden;padding: 0;position: absolute;width: 1px;}#sk-container-id-1 div.sk-estimator {font-family: monospace;background-color: #f0f8ff;border: 1px dotted black;border-radius: 0.25em;box-sizing: border-box;margin-bottom: 0.5em;}#sk-container-id-1 div.sk-estimator:hover {background-color: #d4ebff;}#sk-container-id-1 div.sk-parallel-item::after {content: "";width: 100%;border-bottom: 1px solid gray;flex-grow: 1;}#sk-container-id-1 div.sk-label:hover label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-1 div.sk-serial::before {content: "";position: absolute;border-left: 1px solid gray;box-sizing: border-box;top: 0;bottom: 0;left: 50%;z-index: 0;}#sk-container-id-1 div.sk-serial {display: flex;flex-direction: column;align-items: center;background-color: white;padding-right: 0.2em;padding-left: 0.2em;position: relative;}#sk-container-id-1 div.sk-item {position: relative;z-index: 1;}#sk-container-id-1 div.sk-parallel {display: flex;align-items: stretch;justify-content: center;background-color: white;position: relative;}#sk-container-id-1 div.sk-item::before, #sk-container-id-1 div.sk-parallel-item::before {content: "";position: absolute;border-left: 1px solid gray;box-sizing: border-box;top: 0;bottom: 0;left: 50%;z-index: -1;}#sk-container-id-1 div.sk-parallel-item {display: flex;flex-direction: column;z-index: 1;position: relative;background-color: white;}#sk-container-id-1 div.sk-parallel-item:first-child::after {align-self: flex-end;width: 50%;}#sk-container-id-1 div.sk-parallel-item:last-child::after {align-self: flex-start;width: 50%;}#sk-container-id-1 div.sk-parallel-item:only-child::after {width: 0;}#sk-container-id-1 div.sk-dashed-wrapped {border: 1px dashed gray;margin: 0 0.4em 0.5em 0.4em;box-sizing: border-box;padding-bottom: 0.4em;background-color: white;}#sk-container-id-1 div.sk-label label {font-family: monospace;font-weight: bold;display: inline-block;line-height: 1.2em;}#sk-container-id-1 div.sk-label-container {text-align: center;}#sk-container-id-1 div.sk-container {/* jupyter's `normalize.less` sets `[hidden] { display: none; }` but bootstrap.min.css set `[hidden] { display: none !important; }` so we also need the `!important` here to be able to override the default hidden behavior on the sphinx rendered scikit-learn.org. See: https://github.com/scikit-learn/scikit-learn/issues/21755 */display: inline-block !important;position: relative;}#sk-container-id-1 div.sk-text-repr-fallback {display: none;}</style><div id="sk-container-id-1" class="sk-top-container"><div class="sk-text-repr-fallback"><pre>IsolationForest(contamination=0.1)</pre><b>In a Jupyter environment, please rerun this cell to show the HTML representation or trust the notebook. <br />On GitHub, the HTML representation is unable to render, please try loading this page with nbviewer.org.</b></div><div class="sk-container" hidden><div class="sk-item"><div class="sk-estimator sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-1" type="checkbox" checked><label for="sk-estimator-id-1" class="sk-toggleable__label sk-toggleable__label-arrow">IsolationForest</label><div class="sk-toggleable__content"><pre>IsolationForest(contamination=0.1)</pre></div></div></div></div></div>




```python
# Fit the model on your preprocessed data
required_df = df[['uri_len','uri_segs','uri_s_chars','uri_res_chars','uri_q_params','uri_maxq_len']]
model.fit(required_df)
```




<style>#sk-container-id-2 {color: black;}#sk-container-id-2 pre{padding: 0;}#sk-container-id-2 div.sk-toggleable {background-color: white;}#sk-container-id-2 label.sk-toggleable__label {cursor: pointer;display: block;width: 100%;margin-bottom: 0;padding: 0.3em;box-sizing: border-box;text-align: center;}#sk-container-id-2 label.sk-toggleable__label-arrow:before {content: "▸";float: left;margin-right: 0.25em;color: #696969;}#sk-container-id-2 label.sk-toggleable__label-arrow:hover:before {color: black;}#sk-container-id-2 div.sk-estimator:hover label.sk-toggleable__label-arrow:before {color: black;}#sk-container-id-2 div.sk-toggleable__content {max-height: 0;max-width: 0;overflow: hidden;text-align: left;background-color: #f0f8ff;}#sk-container-id-2 div.sk-toggleable__content pre {margin: 0.2em;color: black;border-radius: 0.25em;background-color: #f0f8ff;}#sk-container-id-2 input.sk-toggleable__control:checked~div.sk-toggleable__content {max-height: 200px;max-width: 100%;overflow: auto;}#sk-container-id-2 input.sk-toggleable__control:checked~label.sk-toggleable__label-arrow:before {content: "▾";}#sk-container-id-2 div.sk-estimator input.sk-toggleable__control:checked~label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-2 div.sk-label input.sk-toggleable__control:checked~label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-2 input.sk-hidden--visually {border: 0;clip: rect(1px 1px 1px 1px);clip: rect(1px, 1px, 1px, 1px);height: 1px;margin: -1px;overflow: hidden;padding: 0;position: absolute;width: 1px;}#sk-container-id-2 div.sk-estimator {font-family: monospace;background-color: #f0f8ff;border: 1px dotted black;border-radius: 0.25em;box-sizing: border-box;margin-bottom: 0.5em;}#sk-container-id-2 div.sk-estimator:hover {background-color: #d4ebff;}#sk-container-id-2 div.sk-parallel-item::after {content: "";width: 100%;border-bottom: 1px solid gray;flex-grow: 1;}#sk-container-id-2 div.sk-label:hover label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-2 div.sk-serial::before {content: "";position: absolute;border-left: 1px solid gray;box-sizing: border-box;top: 0;bottom: 0;left: 50%;z-index: 0;}#sk-container-id-2 div.sk-serial {display: flex;flex-direction: column;align-items: center;background-color: white;padding-right: 0.2em;padding-left: 0.2em;position: relative;}#sk-container-id-2 div.sk-item {position: relative;z-index: 1;}#sk-container-id-2 div.sk-parallel {display: flex;align-items: stretch;justify-content: center;background-color: white;position: relative;}#sk-container-id-2 div.sk-item::before, #sk-container-id-2 div.sk-parallel-item::before {content: "";position: absolute;border-left: 1px solid gray;box-sizing: border-box;top: 0;bottom: 0;left: 50%;z-index: -1;}#sk-container-id-2 div.sk-parallel-item {display: flex;flex-direction: column;z-index: 1;position: relative;background-color: white;}#sk-container-id-2 div.sk-parallel-item:first-child::after {align-self: flex-end;width: 50%;}#sk-container-id-2 div.sk-parallel-item:last-child::after {align-self: flex-start;width: 50%;}#sk-container-id-2 div.sk-parallel-item:only-child::after {width: 0;}#sk-container-id-2 div.sk-dashed-wrapped {border: 1px dashed gray;margin: 0 0.4em 0.5em 0.4em;box-sizing: border-box;padding-bottom: 0.4em;background-color: white;}#sk-container-id-2 div.sk-label label {font-family: monospace;font-weight: bold;display: inline-block;line-height: 1.2em;}#sk-container-id-2 div.sk-label-container {text-align: center;}#sk-container-id-2 div.sk-container {/* jupyter's `normalize.less` sets `[hidden] { display: none; }` but bootstrap.min.css set `[hidden] { display: none !important; }` so we also need the `!important` here to be able to override the default hidden behavior on the sphinx rendered scikit-learn.org. See: https://github.com/scikit-learn/scikit-learn/issues/21755 */display: inline-block !important;position: relative;}#sk-container-id-2 div.sk-text-repr-fallback {display: none;}</style><div id="sk-container-id-2" class="sk-top-container"><div class="sk-text-repr-fallback"><pre>IsolationForest(contamination=0.1)</pre><b>In a Jupyter environment, please rerun this cell to show the HTML representation or trust the notebook. <br />On GitHub, the HTML representation is unable to render, please try loading this page with nbviewer.org.</b></div><div class="sk-container" hidden><div class="sk-item"><div class="sk-estimator sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-2" type="checkbox" checked><label for="sk-estimator-id-2" class="sk-toggleable__label sk-toggleable__label-arrow">IsolationForest</label><div class="sk-toggleable__content"><pre>IsolationForest(contamination=0.1)</pre></div></div></div></div></div>




```python
# Predict anomalies
df['scores']=model.decision_function(required_df)
df['anomaly']=model.predict(required_df)
df.head(20)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>http.request.uri</th>
      <th>uri_len</th>
      <th>uri_segs</th>
      <th>uri_s_chars</th>
      <th>uri_res_chars</th>
      <th>uri_q_params</th>
      <th>uri_maxq_len</th>
      <th>scores</th>
      <th>anomaly</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>/dvwa/config/</td>
      <td>13</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0.090492</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>/dvwa/docs/</td>
      <td>11</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0.087122</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>/dvwa/external/</td>
      <td>15</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0.096042</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>/mutillidae/ajax/</td>
      <td>17</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0.100049</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>/mutillidae/classes/</td>
      <td>20</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0.102356</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>/mutillidae/data/</td>
      <td>17</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0.100049</td>
      <td>1</td>
    </tr>
    <tr>
      <th>6</th>
      <td>/mutillidae/documentation/</td>
      <td>26</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0.102650</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7</th>
      <td>/mutillidae/images/</td>
      <td>19</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0.101738</td>
      <td>1</td>
    </tr>
    <tr>
      <th>8</th>
      <td>/mutillidae/includes/</td>
      <td>21</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0.104347</td>
      <td>1</td>
    </tr>
    <tr>
      <th>9</th>
      <td>/mutillidae/javascript/</td>
      <td>23</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0.107356</td>
      <td>1</td>
    </tr>
    <tr>
      <th>10</th>
      <td>/mutillidae/passwords/</td>
      <td>22</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0.106141</td>
      <td>1</td>
    </tr>
    <tr>
      <th>11</th>
      <td>/mutillidae/phpmyadmin/</td>
      <td>23</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0.107356</td>
      <td>1</td>
    </tr>
    <tr>
      <th>12</th>
      <td>/mutillidae/styles/</td>
      <td>19</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0.101738</td>
      <td>1</td>
    </tr>
    <tr>
      <th>13</th>
      <td>/mutillidae/test/</td>
      <td>17</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0.100049</td>
      <td>1</td>
    </tr>
    <tr>
      <th>14</th>
      <td>/mutillidae/webservices/</td>
      <td>24</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0.104205</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15</th>
      <td>/mutillidae/phpmyadmin/examples/</td>
      <td>32</td>
      <td>3</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0.138496</td>
      <td>1</td>
    </tr>
    <tr>
      <th>16</th>
      <td>/mutillidae/phpmyadmin/js/</td>
      <td>26</td>
      <td>3</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0.145323</td>
      <td>1</td>
    </tr>
    <tr>
      <th>17</th>
      <td>/mutillidae/phpmyadmin/locale/</td>
      <td>30</td>
      <td>3</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0.140898</td>
      <td>1</td>
    </tr>
    <tr>
      <th>18</th>
      <td>/mutillidae/phpmyadmin/setup/</td>
      <td>29</td>
      <td>3</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0.141864</td>
      <td>1</td>
    </tr>
    <tr>
      <th>19</th>
      <td>/mutillidae/phpmyadmin/themes/</td>
      <td>30</td>
      <td>3</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0.140898</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
anomaly=df[df['anomaly']==-1]
anomaly.head(50)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>http.request.uri</th>
      <th>uri_len</th>
      <th>uri_segs</th>
      <th>uri_s_chars</th>
      <th>uri_res_chars</th>
      <th>uri_q_params</th>
      <th>uri_maxq_len</th>
      <th>scores</th>
      <th>anomaly</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>20</th>
      <td>/assets/</td>
      <td>8</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.031069</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>21</th>
      <td>/cgi-bin/</td>
      <td>9</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.017994</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>22</th>
      <td>/evil/</td>
      <td>6</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.034518</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>23</th>
      <td>/gallery2/</td>
      <td>10</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.023963</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>26</th>
      <td>/icon/</td>
      <td>6</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.034518</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>27</th>
      <td>/images/</td>
      <td>8</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.031069</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>28</th>
      <td>/javascript/</td>
      <td>12</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.020447</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>29</th>
      <td>/joomla/</td>
      <td>8</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.031069</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>30</th>
      <td>/phpBB2/</td>
      <td>8</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.031069</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>31</th>
      <td>/phpmyadmin/</td>
      <td>12</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.020447</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>32</th>
      <td>/test/</td>
      <td>6</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.034518</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>33</th>
      <td>/test/</td>
      <td>6</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.034518</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>34</th>
      <td>/wordpress/</td>
      <td>11</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.021533</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>54</th>
      <td>/</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.035260</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>55</th>
      <td>/</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.035260</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>56</th>
      <td>/</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.035260</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>59</th>
      <td>/</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.035260</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>61</th>
      <td>/</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.035260</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>66</th>
      <td>/phpBB2/login.php?redirect=admin/&amp;amp;sid=eb3e...</td>
      <td>74</td>
      <td>2</td>
      <td>6</td>
      <td>1</td>
      <td>2</td>
      <td>32</td>
      <td>-0.164158</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>80</th>
      <td>/nmaplowercheck1685892421</td>
      <td>25</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.005327</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>81</th>
      <td>/</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.035260</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>82</th>
      <td>/nmaplowercheck1685892421</td>
      <td>25</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.005327</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>83</th>
      <td>/nmaplowercheck1685892421</td>
      <td>25</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.005327</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>84</th>
      <td>/</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.035260</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>85</th>
      <td>/</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.035260</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>91</th>
      <td>/HNAP1</td>
      <td>6</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.034518</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>92</th>
      <td>/HNAP1</td>
      <td>6</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.034518</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>95</th>
      <td>/HNAP1</td>
      <td>6</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.034518</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>99</th>
      <td>/wordpress/wp-login.php?redirect_to=%2Fwordpre...</td>
      <td>62</td>
      <td>2</td>
      <td>9</td>
      <td>1</td>
      <td>1</td>
      <td>20</td>
      <td>-0.110149</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>102</th>
      <td>/</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.035260</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>103</th>
      <td>/</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.035260</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>104</th>
      <td>/</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.035260</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>105</th>
      <td>/</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.035260</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>106</th>
      <td>/</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.035260</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>113</th>
      <td>/randomfile1</td>
      <td>12</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.020447</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>114</th>
      <td>/frand2</td>
      <td>7</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.030701</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>115</th>
      <td>/.bash_history</td>
      <td>14</td>
      <td>1</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.022261</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>116</th>
      <td>/.bashrc</td>
      <td>8</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.021757</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>117</th>
      <td>/.cache</td>
      <td>7</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.021395</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>118</th>
      <td>/.config</td>
      <td>8</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.021757</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>119</th>
      <td>/.cvs</td>
      <td>5</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.025407</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>120</th>
      <td>/.cvsignore</td>
      <td>11</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.012127</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>121</th>
      <td>/.forward</td>
      <td>9</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.017994</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>123</th>
      <td>/.history</td>
      <td>9</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.017994</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>124</th>
      <td>/.hta</td>
      <td>5</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.025407</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>125</th>
      <td>/.hta_</td>
      <td>6</td>
      <td>1</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.036611</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>126</th>
      <td>/.htaccess</td>
      <td>10</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.014515</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>127</th>
      <td>/.htaccess_</td>
      <td>11</td>
      <td>1</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.028929</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>128</th>
      <td>/.htpasswd</td>
      <td>10</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.014515</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>129</th>
      <td>/.htpasswd_</td>
      <td>11</td>
      <td>1</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.028929</td>
      <td>-1</td>
    </tr>
  </tbody>
</table>
</div>




```python

```
