' EnumServers.vbs
' VBScript program to enumerate all servers in the domain.
'
' ----------------------------------------------------------------------
' Copyright (c) 2002 Richard L. Mueller
' Hilltop Lab web site - http://www.rlmueller.net
' Version 1.0 - November 10, 2002
' Version 1.1 - February 19, 2003 - Standardize Hungarian notation.
' Version 1.2 - March 11, 2003 - Remove SearchScope property.
' Version 2.0 - February 9, 2004 - Find computers with server operating
'                                  systems.
' Version 2.1 - October 26, 2009 - Use ADO to filter on server OS's.
'
' Program enumerates the Distinguished Name of all computer objects that
' have the string "server" in the operating System attribute.
'
' You have a royalty-free right to use, modify, reproduce, and
' distribute this script file in any way you find useful, provided that
' you agree that the copyright owner above has no warranty, obligations,
' or liability for such use.

Option Explicit

Dim objRootDSE, strDNSDomain, adoConnection, adoCommand, strQuery
Dim adoRecordset, strComputerDN, strBase, strFilter, strAttributes

' Determine DNS domain name from RootDSE object.
Set objRootDSE = GetObject("LDAP://RootDSE")
strDNSDomain = objRootDSE.Get("defaultNamingContext")

' Use ADO to search Active Directory for all computers.
Set adoCommand = CreateObject("ADODB.Command")
Set adoConnection = CreateObject("ADODB.Connection")
adoConnection.Provider = "ADsDSOObject"
adoConnection.Open "Active Directory Provider"
adoCommand.ActiveConnection = adoConnection

' Search entire domain.
strBase = "<LDAP://" & strDNSDomain & ">"

' Filter on computer objects with server operating system.
strFilter = "(&(objectCategory=computer)(operatingSystem=*server*))"

' Comma delimited list of attribute values to retrieve.
strAttributes = "distinguishedName"

' Construct the LDAP syntax query.
strQuery = strBase & ";" & strFilter & ";" & strAttributes & ";subtree"

adoCommand.CommandText = strQuery
adoCommand.Properties("Page Size") = 100
adoCommand.Properties("Timeout") = 30
adoCommand.Properties("Cache Results") = False

Set adoRecordset = adoCommand.Execute

' Enumerate computer objects with server operating systems.
Do Until adoRecordset.EOF
    strComputerDN = adoRecordset.Fields("distinguishedName").Value
    Wscript.Echo strComputerDN
    adoRecordset.MoveNext
Loop

' Clean up.
adoRecordset.Close
adoConnection.Close

Wscript.Echo "Done"
