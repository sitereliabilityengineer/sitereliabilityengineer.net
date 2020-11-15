---
title: Python’s map power – newbies
author: sitereal
type: post
date: 2017-11-04T10:04:23+00:00
url: /2017/11/04/pythons-map-power-newbies/
categories:
  - Python
  - Python tips

---
## There are some methods that I love, one of them is map. Surely you already know!

Imagine that you want to read the line cpu in the file /proc/stat

The file looks similar to this:

cpu 5513 0 7271 2832299 15350 1 1327 0 0 0

cpu1 xxx xxx xxx xxx xxx xxx xxx

  1. We open the file with open and assign it to a file object with (&#8216;as&#8217;)
  2. Loop the file object (st)
  3. If the line startswith(&#8216;cpu &#8216;) then we continue. (I left a space on the right of cpu, because in the file also we have cpu1, cpu2, and I only need &#8216;cpu&#8217;.
  4. I assign the result of map. We split the line starting in index 1 and we map the elements executing float in every element.

&nbsp;

<div class="codecolorer-container python blackboard" style="overflow:auto;white-space:nowrap;width:650px;">
  <table cellspacing="0" cellpadding="0">
    <tr>
      <td class="line-numbers">
        <div>
          1<br />2<br />3<br />4<br />5<br />
        </div>
      </td>
      
      <td>
        <div class="python codecolorer">
          <span class="kw1">with</span> <span class="kw2">open</span><span class="br0">&#40;</span><span class="st0">'/proc/stat'</span><span class="sy0">,</span> <span class="st0">'r'</span><span class="br0">&#41;</span> <span class="kw1">as</span> st:<br /> &nbsp; &nbsp; <span class="kw1">for</span> line <span class="kw1">in</span> st:<br /> &nbsp; &nbsp; &nbsp; &nbsp; <span class="kw1">if</span> line.<span class="me1">startswith</span><span class="br0">&#40;</span><span class="st0">'cpu '</span><span class="br0">&#41;</span>:<br /> &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; line_split <span class="sy0">=</span> <span class="kw2">map</span><span class="br0">&#40;</span><span class="kw2">float</span><span class="sy0">,</span> line.<span class="me1">strip</span><span class="br0">&#40;</span><span class="br0">&#41;</span>.<span class="me1">split</span><span class="br0">&#40;</span><span class="br0">&#41;</span><span class="br0">&#91;</span><span class="nu0">1</span>:<span class="br0">&#93;</span><span class="br0">&#41;</span><br /> &nbsp; &nbsp; <span class="kw1">print</span> line_split
        </div>
      </td>
    </tr>
  </table>
</div>

_Other way to do it:  with a list comprehension:_

<div class="codecolorer-container text blackboard" style="overflow:auto;white-space:nowrap;width:650px;">
  <table cellspacing="0" cellpadding="0">
    <tr>
      <td class="line-numbers">
        <div>
          1<br />
        </div>
      </td>
      
      <td>
        <div class="text codecolorer">
          line_split = [ int(x) for x in line.strip().split()[1:] ]
        </div>
      </td>
    </tr>
  </table>
</div>