def setType(kwargs):
    parm = kwargs['node'].path() + "/params"
    state = int(kwargs['script_value'])+1
    hou.parm(parm).set(state)

def getDriver(node):
    renderNode = None
    if len(node.inputs()) > 0:
        renderNode = node.inputs()[0]
    if renderNode is None:
        renderPath = node.parm("driver").eval()
        renderNode = node.node(renderPath)
    return renderNode
    
def getSettings(node):
    sets = []
    dim = node.parm('params').eval()
    for i in range(dim):
        i = str(i+1)
        
        npath = node.parm('node'+i).eval()
        nchan = node.parm('chan'+i).eval()
        chan  = npath + "/" + nchan
        val   = hou.parm(chan).eval()
        
        sets.append( (chan, val) )
    return sets

def mergeWedge(allwedge, wedge):
    result = []
    for w in wedge:
        for a in allwedge[:]:
            a = a[:]
            a.append(w)
            result.append(a)
    return result
    
def collectWedges(node):
    allWedges = [[]]
    
    dim = node.parm('params').eval()
    for i in range(dim):
        i = str(i+1)
        
        title = node.parm('title'+i).eval()
        npath = node.parm('node'+i).eval()
        nchan = node.parm('chan'+i).eval()
        chan  = npath + "/" + nchan
        val   = node.parm('values'+i).eval()
        
        wedges = []
        vals  = map(str, val.split())
        for v in vals:
            # add checks if items are valid
            #   check if final channel path is valid
            #   check if values are all numeric
            wedges.append( (title, chan, v) )
        allWedges = mergeWedge(allWedges, wedges)
    return allWedges
    
def setenvvariable(var, val):
    hou.hscript("set %s = %s" % (var, val))
    hou.hscript("varchange")
    
def applyWedge(node):
    driver    = getDriver(node) # render driver
    sets      = getSettings(node) # save original settings
    allWedges = collectWedges(node) # wedges array
    
    # check if rendernode is valid
    
    # Disable background rendering
    fgparm = None
    if node.parm("blockbackground").eval() == 1:
        fgparm = driver.parm("soho_foreground")
    
    if fgparm is not None:
        oldfgval = fgparm.eval()
        fgparm.set(True)
        
    wedgeNum = 0
    for wedgePair in allWedges:
        titleName = ""
        valueName = ""
        for wedge in wedgePair:
            title = wedge[0]
            param = wedge[1]
            value = wedge[2]    
            
            hou.parm(param).set(value)
            
            titleName += "_" + str(wedge[0])
            valueName += "_" + str(wedge[2])
            setenvvariable("WEDGENUM", str(wedgeNum))
            wedgeNum += 1
        
        setenvvariable("WEDGE", titleName)
        setenvvariable("WEDGEVAL", valueName)        
        driver.render()        
        titleName = ""
        valueName = ""
    
    # Restore background rendering
    if fgparm is not None:
        fgparm.set(oldfgval)
        
    # Revert to original settings
    for s in sets:
        hou.parm(s[0]).set(s[1])
    
    setenvvariable("WEDGE", "")
    setenvvariable("WEDGENUM", "-1")
    setenvvariable("WEDGEVAL", "")
    
    