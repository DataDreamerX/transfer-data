"use client"

import { Avatar, AvatarFallback } from "@/components/ui/avatar"
import { Button } from "@/components/ui/button"
import { Card, CardContent } from "@/components/ui/card"
import { Edit3, Copy, Volume2, ThumbsUp, ThumbsDown, RotateCcw, Share, MoreHorizontal } from "lucide-react"

interface ChatCardProps {
  title?: string
  message: string
  avatarColor?: string
  onEdit?: () => void
  onCopy?: () => void
  onSpeak?: () => void
  onLike?: () => void
  onDislike?: () => void
  onRefresh?: () => void
  onShare?: () => void
  onMore?: () => void
}

export function ChatCard({
  title = "gpt-4o",
  message,
  avatarColor = "bg-purple-500",
  onEdit,
  onCopy,
  onSpeak,
  onLike,
  onDislike,
  onRefresh,
  onShare,
  onMore,
}: ChatCardProps) {
  return (
    <Card className="w-full max-w-2xl">
      <CardContent className="p-4">
        <div className="flex items-start gap-3">
          <Avatar className={`w-8 h-8 ${avatarColor} text-white text-xs font-medium`}>
            <AvatarFallback className={`${avatarColor} text-white`}>{title.slice(0, 2).toUpperCase()}</AvatarFallback>
          </Avatar>

          <div className="flex-1 space-y-3">
            <div>
              <h3 className="font-medium text-sm mb-1">{title}</h3>
              <p className="text-sm text-muted-foreground leading-relaxed">{message}</p>
            </div>

            <div className="flex items-center gap-1">
              <Button variant="ghost" size="sm" className="h-8 w-8 p-0 hover:bg-muted" onClick={onEdit}>
                <Edit3 className="h-4 w-4" />
              </Button>

              <Button variant="ghost" size="sm" className="h-8 w-8 p-0 hover:bg-muted" onClick={onCopy}>
                <Copy className="h-4 w-4" />
              </Button>

              <Button variant="ghost" size="sm" className="h-8 w-8 p-0 hover:bg-muted" onClick={onSpeak}>
                <Volume2 className="h-4 w-4" />
              </Button>

              <Button variant="ghost" size="sm" className="h-8 w-8 p-0 hover:bg-muted" onClick={onLike}>
                <ThumbsUp className="h-4 w-4" />
              </Button>

              <Button variant="ghost" size="sm" className="h-8 w-8 p-0 hover:bg-muted" onClick={onDislike}>
                <ThumbsDown className="h-4 w-4" />
              </Button>

              <Button variant="ghost" size="sm" className="h-8 w-8 p-0 hover:bg-muted" onClick={onRefresh}>
                <RotateCcw className="h-4 w-4" />
              </Button>

              <Button variant="ghost" size="sm" className="h-8 w-8 p-0 hover:bg-muted" onClick={onShare}>
                <Share className="h-4 w-4" />
              </Button>

              <Button variant="ghost" size="sm" className="h-8 w-8 p-0 hover:bg-muted" onClick={onMore}>
                <MoreHorizontal className="h-4 w-4" />
              </Button>
            </div>
          </div>
        </div>
      </CardContent>
    </Card>
  )
}
